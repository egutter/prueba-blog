---
author: Facundo Spagnuolo
categories:
- ruby
- rails
- best practices
comments: true
date: 2015-11-25T15:28:11Z
title: Ruby On Pains
url: /2015/11/25/ruby-on-pains/
---

Have you ever heard about the *Rails way*?
I would like to introduce some pains that I've seen and keep seeing in all the Rails projects due to the *Rails way*...

<!--more-->

## ActiveRecord
How many times do we need to find an object in a Rails project? How many times do we create, update or delete objects from the DB? Many times, isn't it? We are used to repeat these actions in all our ActiveRecord classes. We do it like monkeys, never ask for a timeout and think about it. It seems to be something *normal* when we use this framework, it's the famous *Rails way*. It's *normal* for us to have a class that answers `find`, `update`, `save`, etc, but we never think if it’s right and whether there could be a better way of doing it.

It's typical to find this kind of logic in a Rails project, see the example of this controller:

{{< highlight ruby >}}
class UserController
  def index
    @users = User.all
  end
  
  def show
    @user = User.find(params[:id])
  end
    
  def create
    @user = User.create(user_params)
  ...
  private
 
  def user_params
    params.require(:user).permit(...)
  end
  ...
{{< / highlight >}}

The OO paradigm defines, among other things, that a class has two main responsibilities. The first one is defining the behavior for the objects of that class, and the second one is creating instances of it. This means that all objects that are instances of the same class can perform the same actions. So, when did we talk about persisting? We didn't say anything about classes having the responsibility of writing to the database or executing a query in order to load some instances in memory. Have you ever thought what are we doing to our classes in order to answer those kind of messages? Basically inheriting from `ActiveRecord::Base`, which is a strong association that defines a rigid behavior that doesn't follow the paradigm principles. Look at this alternative:

{{< highlight ruby >}}
class UserController
  def index
    @users = UserRecords.find_all
  end
  
  def show
    @user = UserRecords.find(params[:id])
  end
  ...
{{< / highlight >}}

Definitely our business domain models shouldn't be coupled to the way we persist them. The main reason is both belong to different domains, having different kind of responsibilities. Moreover, have you ever thought about changing Rails for another framework? Our models shouldn't change, our code shouldn't change much. We should only need to change those entities in charge of persistence. But if those entities are the same that model our business domain, probably we are in trouble.


## Validations

Have you ever thought about invalid objects? Moreover, have you ever thought if it make sense to think about valid objects? Well, in real life we don't have invalid entities, we don't have invalid persons, or invalid cars, it would be ridiculous. But, since the external world interacts with computer systems, for example someone using a web application, we can always do it in a bad way. We can fill forms incorrectly, e.g. filling a telephone input field with my name. However this doesn't mean that invalid objects should exist. We can have validations or rules that need to be satisfied in order to process that form and create a new user in our system.

Let me be more specific. In a Rails project we can probably find controllers like...

{{< highlight ruby >}}
def create
  @user = User.create(params[:user])
  if @user.valid?
    redirect_to show_path(@user)
  else
    render :create, errors: @user.errors
  end
end
{{< / highlight >}}

We usually ask an object if it's valid because, I suppose, it's the *Rails way*. As I said, it's ridiculous, I'm sure that User class is full of `validates` and we still can create invalid instances.

Suppose we are in 1930, and you go to a club asking for a sign up. The help desk gives you the users record book and asks you to write your enrollment, then the help desk checks whether you have filled the enrollment correctly. If you did it wrong, you will be asked to fix it. Well, I'm sure that book would be full of corrections. Wouldn't it be better to have a sign up form that the helpdesk uses to complete an enrollment that we know is correct?

The problem here is that we are modeling a user when we don't have to. We are omitting something in the middle, the form. We aren't modeling that, take a look at this short example:

{{< highlight ruby >}}
def create
  form = UserEnrollmentForm.new(user_params)
  @user = UserEnroller.new.call(form)
  redirect_to show_path(@user)
rescue UserEnrollmentError => e
  render :create, form: form, error: e
end
{{< / highlight >}}

Here we are just delegating the responsibility of deciding whether to create or not a User based on an input form. Again, have in mind the given implementation is not part of the scope, we can discuss multiple ways of doing this. The main thing is, we should never have invalid objects in our system. If something goes wrong while creating or modifying an instance, I would like to be notified asap - In Rails, use the bang always!

## Ruby Coals

I like using that word just to refer to awful gems. What happens when you grab a coal? Your hands get dirty right? Well, this is what I feel with some gems when I start using them in a project. Also a coal is the primitive of a gem and it really feels like we are not progressing when we use that kind. The fact is that the *Rails way* is awesome cause we can find some functionality that someone has already built, installing that gem, and voilá. 

Let's see some examples. We can start with this pagination coal...

{{< highlight ruby >}}
class Post
  self.per_page = 10
end

class PostsController
  ...
  @posts = Post.paginate(page: params[:page])
  ...
end
{{< / highlight >}}

I don't like that way of doing things, we are breaking the OO design rule about responsibilities and not coupling things that belong to different domains. Here, we are coupling our `Post` model with the idea of paginating them for a view, that's crazy! We should find a better way, such as modeling a paginator instead of having our classes answering messages like `per_page` or `paginate`:

{{< highlight ruby >}}
class PostsController
  ...
  paginator = Paginator.new(PostsBook.find_all, per_page: POSTS_PER_PAGE)
  posts = paginator.call(page: params[:page])
  ...
end
{{< / highlight >}}

Now take a look at this filtering coal...

{{< highlight ruby >}}
class Student
  scope :with_country_id, -> (country_id) { … }
  scope :sorted_by, -> (field) { ... }
  scope :search_query, ...
      
  filterrific(
    default_filter_params: { sorted_by: 'created_at_desc' },
    available_filters: [:sorted_by, :search_query, :with_country_id])
  ...
end

class StudentController
  def index
    @filterrific = initialize_filterrific(Student, params[:filterrific]) or return
    @students = @filterrific.find.page(params[:page])
    respond_to do |format|
      format.html
      format.js
    end
  end
  ...
end
{{< / highlight >}}

That's really good! Our models become a storage of filtering configuration! Seriously, the filtering functionallity that we offer in a view has nothing to do in our `Student` model. Does a student need to know about filtering configuration? One more time, we are coupling. Here is another simple way of doing this: 

{{< highlight ruby >}}
class StudentController
  ...
    students_filter = StudentsFilter.new(params[:filtering])
    @students = StudentRecords.with_filter(filter).find_all
  ...
end
{{< / highlight >}}

Again, let me remeber that the examples shown above are just ilustrative, we are not discussing the implementation, but the approach we are choosing. My suggestion is, let's think twice before adding this kind of coals as your code will get dirty and removing or refactoring this kind of functionality to another place will be a pain in the ass.

## Conclusion

In my opinion, Rails is a good framework for a kick-off. It’s easy to write the firsts test cases, implement the idea, and deploy it. But, what happens when the application starts to grow? Sometimes we lose our mind trying to get things out faster, and we shouldn't forget the importance of designing good models, otherwise implementing the next feature becomes a headache. Also you will always find multiple gems to solve your problem, but think the way you are going to implement such thing, most of them make your models dirty. Always remember that it is key to understand the essence of the objects in the reality's domain to keep it on our code. 