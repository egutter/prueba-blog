---
author: Emilio Gutter
categories:
- refactoring
comments: true
date: 2014-02-01T00:00:00Z
title: Refactoring legacy code story
url: /2014/02/01/refactoring-legacy-code-story/
---

## Introduction

Is the following story familiar? A client nocks to your door, asking for help
with a software developed by another vendor. He complains about developers
always being late to deadlines and every time a new version is delivered there
are plenty of things that are not working properly. Issues fixed on previous
versions come back as living deads on every new deploy: _something is fixed
here, another thing is broken there_.

<!--more-->

You accept the new challenge, sign an NDA and get access to the code. What's the
first thing you do? Run the tests of course. Surprise, surprise, you find out
you are dealing with a NFT[^1] situation. So, you just setup your environment,
start up the app and navigate it with your browser. After making yourself a bit
familiar with the app's functionality, you feel it's time to take a look at the
code. You poke around a few different classes and you think you are about to hit
a new WTF[^2] per minute record.

If you have been in a similar situation before and you didn't run away from it,
then you have dealt with Legacy Code and you have spent some time finding the
best way to handle it.

## Put the system under test

You want to fix bugs and refactor the code, without test you'll never know when
you're done and you won't know if fixing/refactoring something didn't break
something else. Manually testing the application with every little change is
usually not humanly possible. The bad news is Rome wasn't built in a day, nor
your test harness will be. So, you will still need some level of manual testing
and you will break things in the beginning. However, the only way to ensure at
some point in time, you'll be able to rely on an automated test harness, is to
start building it as soon as possible.

What kind of tests should you start writing? The recommended strategy is to have
a few set of integration/end-to-end tests and focus in a big set of unit or
micro tests[^3]. However given the code is a POS[^4] not only you'll change it
radically in the short future, but more important writing unit or micro test
might be an impossible task.

The internal implementation is (hopefully) going to change soon, whereas the
external behaviour will remain stable in the short/mid-term. Therefore writing
more integration/end-to-end tests in the beginning, although going against the
best practices patterns, is the best way to start. I won't dig deeper into
writing these kind of tests here, but if you're interested about it you can read
this other post I wrote[^5].

## Start refactoring

Let's go through an example. Imagine we have an application similar to Myspace,
and we want to allow users to upload their songs, either on audio or video
format, to share it with other users. Next is a real implementation I had to
deal with of a Controller which handles post requests from the browser:

``` ruby

class SongsController < ApplicationController
  def create
    if params[:song][:as_post]
      @post = true
      params[:song].delete(:as_post)
    end
    @song = current_user.songs.build(params[:song])
    @song.is_portofolio = 1
    if @song.file_type == 'video'
      if validate_video_format == 1
        flash[:error] = 'Some error message.'
      end
    end
  
    respond_to do |format|
      if @song.save
        @success = true
        if params[:song][:genre_ids]
          params[:genre][:genre_ids].each do |genre_id|
          song_genre = SongGenre.new(:song_id => @song.id, :genre_id => genre_id)
            if song_genre.valid?
              song_genre.save
            else
              @errors += song_genre.errors
            end
          end
          end
        @my_favourites = Song.where(:user_id => current_user.id, :description => "favourite")
        @my_songs = current_user.songs.where(file_type: @song.file_type).paginate(:per_page => 20, :page => params[:page])
        write_activity_stream_log(:current_user, :full_name, :uploaded_song,
          :@song, :title, :create, :upload_song)
        flash[:just_uploaded_song] = true
        unless request.env["HTTP_REFERER"].include?('songs')
          format.html {
            if params[:signup_wizard_form_two]
                redirect_to(signup_wizard_step_three_path, :notice => 'Your song has successfully been uploaded.')
            else
                redirect_to(:back, :notice => 'Your song has successfully been uploaded.')
            end
            }
          format.js
        else
          format.html { redirect_to songs_url), :notice => 'Your song has successfully been uploaded.' }
          format.js
        end
      else
        @success = false
        @my_favourites = Song.where(:user_id => current_user.id, :description => "favourite")
        @my_songs = current_user.songs.where(file_type: @song.file_type).paginate(:per_page => 20, :page => params[:page])
          format.html { redirect_to :back, notice: 'Your song has not uploaded.'}
          format.js
      end
    end
  
  end
end

```

The first step is to understand what's going on. For this purpose
[Extract Method](http://www.refactoring.com/catalog/extractMethod.html) refactor
is your best friend. Our first refactored version might look like this:

``` ruby

class SongsController < ApplicationController
  def create
    save_post_param    
    @song = build_a_new_song
    validate_video_format
  
    respond_to do |format|
      if @song.save
        @success = true
        add_genres_to_song
        load_my_favourites
        load_my_songs
        log_song_uploaded
        flash[:just_uploaded_song] = true
        unless user_is_uploading_from_song_list
          format.html {
            if user_is_in_the_sign_up_wizard_step_two
              redirect_user_to_signup_wizard_step_three
            else
              redirect_user_to_previous_page
            end
            }
          format.js
        else
          format.html { redirect_user_to_song_list }
          format.js
        end
      else
        @success = false
        load_my_favourites
        load_my_songs
        format.html { redirect_user_to_previous_page_with_error_message }
        format.js
      end
    end
  end
end

```

This was just a first level of methods extracted. You can keep extracting more
methods, for example those big chunks of success or error blocks. The second
usual step is to use [Extract Class](http://www.refactoring.com/catalog/extractClass.html)
refactor and move methods to other objects with fewer responsibilities. However,
assuming this codebase has been changed by several people at different stages,
it's very likely that much of the code is not used or is not useful any more,
so we will be refactoring and testing a lot of dead code.

Another design problem from our previous example is handling different use cases
from the same place (i.e. _'users uploads a song from the song list'_; _'user
uploads a song from the signup wizard'_; _'user uploads a video vs an audio'_;
etc.). If we write new controllers for each use case and move the common logic
in some new objects, we can avoid duplicated code while removing complexity and
getting rid of many of the IF-ELSE condition statements.

The approach I usually take, is to start with only one use case, move the
minimum code I think it is required and then continue with the rest of the use
cases. How are we going to know if any code left aside is useful or if we have
introduced any new bugs? As we have written the integration/end-to-end tests
before starting with the refactoring, when all those tests are back to green,
we will then know we are done.

Next is the a version I wrote for the controller related to the use case _'user
uploads a video from the song list'_:

``` ruby

class SongsController < ApplicationController
  
  after_filter :load_my_songs
  
  def save_video
    MusicPortfolio.for(current_user).add_video(new_video_data).
      on_success(method(:handle_success_video_upload)).
      on_error(method(:handle_error_video_upload))
  end
  
  private
  
  def new_video_data
    # Extract video data from params
  end
  
  def load_my_songs
    # Load songs
  end
  
  def handle_success_video_upload(song)
    @song = song
    respond_to do |format|
      format.html {
        redirect_to songs_url, :notice => 'Your song has successfully been uploaded.'
      }
      format.js {
        render "video_upload_success"
      }
    end
  end
  
  def handle_error_video_upload(song, errors)
    @song = song
    respond_to do |format|
      format.html {
        redirect_to :back, notice: 'Your song has not uploaded.'
      }
      format.js {
        render "video_upload_error"
      }
    end
  end
end

```

It's important to notice in the previous example how we got rid of all the
complex IF-ELSE conditions and how the common logic that will be reused in the
other use cases has been moved to the MusicPortfolio object.

## Some final thoughts

After testing-refactoring the application using this strategy, you will end up
with a big suite of integration/end-to-end tests. You won't be able to run this
suite continuously without slowing you down significantly. Assuming, while we
refactored the code, we've been also writing isolated tests for the new objects
we wrote, we can:

* Get rid of some integration tests which are redundant
* Flag the slow/redundant tests to be run only on a nightly basis in the CI server

I usually take the latter approach in the beginning, until the integration tests
prove to either be useless or too fragile and break too frequently for unrelated
causes without caughting any real bug.

[^1]: No Fucking Tests situation also known as WTFWYT (What The Fuck Were You Thinking?)
[^2]: What The fuck per minute is a standard measure of code quality very popular in the agile community. Read more [here](http://www.osnews.com/story/19266/WTFs_m)
[^3]: http://martinfowler.com/bliki/TestPyramid.html
[^4]: Piece Of Shit, also known as BPOS (Big Piece Of Shit)
[^5]: End-to-end test patterns article is coming soon
