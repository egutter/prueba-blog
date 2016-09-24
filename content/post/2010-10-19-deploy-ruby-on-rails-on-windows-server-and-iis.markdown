---
author: Emilio Gutter
categories:
- rails
- windows
- iis
comments: true
date: 2010-10-19T00:00:00Z
title: deploy ruby on rails on windows 2008 server and iis 7.0
url: /2010/10/19/deploy-ruby-on-rails-on-windows-server-and-iis/
---

Deploying Ruby On Rails in a Windows environment can be very painful. I couldn't find any blog which really helped me, so I decided to summarize all the necessary steps I followed to successfully deploy a Rails application in a Windows 2008 server from scratch. I won't give much explanation on each step, so if you like to know more about it, at the end of this post you'll find a list of articles I found useful.

<!--more-->

**The environment: **

* Windows 2008 SP2 x64
* SQL Server 2005
* IIS 7.0
* Ruby 1.8.7
* Ruby on Rails 2.3.5
* Mongrel 1.1.5

**Steps:**

1. Install Ruby VM
2. Install Rails: `gem install rails -v '= 2.3.5' --include-dependencies`
3. Install any other dependencies your application requires
4. Install SQL Server adapter: `gem install activerecord-sqlserver-adapter --version 2.3.5`
5. Install ruby-odbc
	a. Download ruby-odbc
	b. Copy the zip's content to `${RUBY_INSTALLATION_DIR}\lib\ruby\site_ruby\1.8\i386-msvcrt`
6. Open the ODBC Data Source Administrator with the command `C:\Windows>SysWOW64\odbcad32.exe` (make sure you use this command instead of using the regular access from the control panel)
7. Create a System DSN using your specific database connection properties
8. Open the command prompt and go to the Rails application root folder
9. Create and initialize your database:
 * `rake db:create RAILS_ENV=production`
 * `rake db:migrate RAILS_ENV=production`
 * `rake db:seed RAILS_ENV=production`
10. Install IIS Application Request Routing and URL Rewrite extensions
	Edit the hosts file located at `C:\Windows\System32\drivers\etc\hosts` and add the following line `127.0.0.1 localhost localhost-1 localhost-2` (add as many localhost-x hostnames as mongrel services you're planning to setup)
11. Open IIS Manager and create a new farm. Add one server for each of the mongrel services you're planning to setup. For each server use a different address (i.e. localhost-1, localhost-2, etc.) and a different port number (i.e. 3000, 3001, etc.)
12. Edit URL Rewrite rules and add the following rules
 **Rule 1**
 * Pattern: ^$
 * Action: Route to Server Farm
 * Path: /main/index
 * Check stop processing subsequent rules

 **Rule 2**
 * Pattern:^([^.]+)$
 * Action: Route to Server Farm
 * Path: /{R:1}
 * Do not check stop processing subsequent rules
13. In the IIS Manager create a new Web Site and in the Path box, type or browse to the directory that contains your Rails public folder.
14. Download this win32 services gem and install it.
15. Download this mongrel-service gem and install it.
16. Setup as many mongrel services as you need
```
mongrel_rails service::install -N my-app-node-1 -e production -p 3000
mongrel_rails service::install -N my-app-node-2 -e production -p 3001
etc..
```
17. Launch the Window Services console from Control Panel. Change the startup type of your mongrel services to Automatic and start each of them.

18. Open a browser and test your application

**Finally, some articles you might find useful:**


[ODBC binding for Ruby](http://www.ch-werner.de/rubyodbc/)

[Create a 32bit DSN on a 64bit operating system](http://www.boche.net/blog/index.php/2009/11/21/create-a-32-bit-vcenter-dsn-on-a-64-bit-operating-system/)

[Ruby on Rails in IIS70 with url rewriter](http://ruslany.net/2008/08/ruby-on-rails-in-iis-70-with-url-rewriter/)

[Define and configure an application request routing server farm](http://learn.iis.net/page.aspx/485/define-and-configure-an-application-request-routing-server-farm/)

[Creating rewrite rules for the url rewrite module](http://learn.iis.net/page.aspx/461/creating-rewrite-rules-for-the-url-rewrite-module/)

[10 steps to get Ruby on Rails running on Windows with IIS FastCGI](http://mvolo.com/blogs/serverside/archive/2007/02/18/10-steps-to-get-Ruby-on-Rails-running-on-Windows-with-IIS-FastCGI.aspx)



