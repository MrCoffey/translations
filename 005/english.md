In last week's episode we created a basic calendar with some stylings, and threw the library into a Rails app. We created a couple of partials to display the `calendar` library. In this episode, we're going to take that further. We're going to add some CoffeeScript so that if an element has a class of `today` and a class of `day`, or a class of `future` and the class of `day`, we're going to make that element clickable through CoffeeScript which will take us to another URL which we'll create today in the Rails app. In that URL you can create an event, and save the event. It'll redirect you back to the calendar and you should see the event inside of the calendar day. Let's write the JavaScript to make that happen. 

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/001.png "calendario")

We're going to start by wrapping the `document` object in some jQuery which gives us access to the `ready` method. When the document has been loaded and jQuery has been loaded, we're going to match for an element that has the `day` class and the `today` class. If those 2 CSS classes exist on the element, we're going to monitor that element for a 'click' action. When that event happens we're going to run another anonymous function. Right now, let's just do an alert 'wired up'. If we click on the div that represents `today` it should give us an alert. Excellent.

~~~coffescript
$(document).ready ->
 $(".day.today").on "click" ->
   alert "wired up"
~~~

Now, I've reloaded the page. Let's see what happens when we click on the 19th. There's our message. jQuery is indeed wired up and our classes are being watched. We don't want to just be able to click today. We also want to be able to click a day in the future, so we need to look for the CSS class of `.day` and `.future`. If we have a div that matches either .day and today, we're going to be able to click that, or if we find a div that has .day and future, we're going to be able click that.

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/002.png "calendario")

Now, I've reloaded this again. I should be able to not just click on the 19th but I should also be able click on the 20th. I should not, however, be able to click on the 18th because that was yesterday. There you have it.

The next step is, we don't really want to do an alert when this happens, we want to send the user to a different place for now to keep things simple. We're going to do `window.location.href`. This is going to equal to `/events/new` which is simply a RESTful Rails route. However, this Rails route does not currently exist. All right, so the page has reloaded again. If I click on the 19th now, it'll try to take us to a route that doesn't exist. Let's create that route in Rails now.

~~~
$(document).ready ->
  $(".day.today, day.future").on "click" ->
    window.location.href="events/new"
~~~

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/003.png "calendario")

To create the route we're going to go into our `Config Routes` file. Inside the Config Routes file we're going to add a new route. This time it's going to be a plural route, and it's going to be called `Events`. We're only going to start out with the action at hand. We only want the `new` action.

~~~
Rails.application.routes.draw do
  resource :calendar, only: %i[show]
  resources :events, only: %i[new create]
end
~~~

If I was now to run `rake routes`, and then through a pipe grep out anything that has to do with `event`, I should get 1 route for a `new event`. Now, if we reload this web browser, we'll get a different error. That error is that the controller doesn't exist. Let's create that next.

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/004.png "calendario")

We're going to create an `events controller`, and I'm not going to use a generator here. I'll just create a file. It's going to be called 'events_controller.rb'. It's going to be a class of `EventsController` in constant class form. It's going to inherit from ApplicationController. Right now, we just need 1 action in it. It's called `new` and in this action we're going to have a new event which is going to be equal to `Event.new`.

~~~
#rubycast_calendar/app/controllers/ 

class EventsController < ApplicationController
    def new
        @event = Event.new
    end
end
~~~

When we reload the page again we're presented with this error, and that's because we don't have a model yet. We're going to create the model now. We're going to do that right here with a generator, `rails g model`. The model is called event. It's going to have 1 attribute and that's name. Since the attribute is a stream we don't need to give it an argument.

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/005.png "calendario")

Now, we're going to migrate the database because by creating a model through a generator we also we a migration that matches the model. Now with the migrated, if we come back to that page and we refresh, we get a new error and that's that the template is missing. This is correct. We need a folder called `/events` inside of our views folder and in there we need a view called `new`. Let's do that.

To create this we're going to create a new folder called `events`. Inside of here we're going to create a new view called `new.html.erb` because the file is going to end up as HTML but we're going to have a pre-proccessor, erb. Put a basic header tag in here just to identify the page. When we reload the page again, we get this page now where it says `New event`. 

It's time to put the form into this page so that when we create a new event when we get redirected back to our calendar we can see the event on the correct day. I'll just do that in line in this file. The object that the the form is going to operate on is that new event in the instance variable we created in the controller. Since we have a `form_for` we have a block the `f` variable. On my end tag I'm going to use a hyphen so that it doesn't print out an empty line to my HTML.

~~~
<h1>New event</h1>

<%= form_for @event do |f| %>
  <%= f.text_field :name %>
  <%= f.submit %>
<%- end %>
~~~

Now, I'm going to need a `text input` field and it's going to be called `Name` because that's attribute on the model. We're going to need a submit button. We get our form so let's fill out that form. We're going to give it just a name of `Test 2`. Now, we get `Unknown action, 'create' could not to be found`. I'm expecting this error as well so let's go ahead and fix it.

To fix this error, we're going to need to open up the controller and it's going to need another action. It's called `create`. The create action we're going to set an @event instance variable to `Event.new`, very similar to how we did in the new action. However, now we're going to have parameter from a form. Because we're going to have parameters from a form, we need to whitelist those parameters to keep our site from being attacked.

~~~
class EventsController < ApplicationController
  def new
    @event = Event.new
  end

  def create
    @event = Event.new(event_params)
  end

private

  def event_params
    params.require(:event).permit(:name)
  end
end
~~~

Since the name of the model is `Event` our method name for the secure parameters is going to be called `event_params` and that method simply looks like this; we start with the params object that comes from the controller and we are going to require the key of `event` because that's the model name. Then we're going to permit some attributes, in this case, just name. If you leave out the permission here in some older versions of Rails it just becomes nil and you're not notified. In some of the newer versions you're actually notified.


Let's rerun this now. I'm just going to re-post the form here. This time we get a `Template missing error` and that's because at the end of our method this is a `create`, it's a `post`, which shouldn't be rendering HTML from a post. We need to redirect if the action is successful and, we need to re-render if the action is not. To do that, we need to up into our `Create` action.

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/006.png "calendario")

We're going to put an `if` check here to see if the event could be saved. If it can, we're going to redirect ourselves to the `calendar path`. If we cannot, we're going to re-render the form which is at `new`. If we refresh now, re-post our form, it's a successful action. The event was created so we've been sent on to the calendar. However, we don't have the code in the calendar yet to pass the dates that we clicked into that form that we just created. Let's do that now.

~~~
class EventsController < ApplicationController
  def new
    @event = Event.new
  end

  def create
    @event = Event.new(event_params)
    if @event.save
      redirect_to calendar_path
    else
      render :new
    end
  end

private

  def event_params
    params.require(:event).permit(:name)
  end
end
~~~

I have my CoffeeScript file for the calendar open and I also have the partial that represents a day open. We're going to down into the partial that represents a day and we're going to add a new data attribute to consume with the JavaScript. This is going to be called `date`, and it's going to have the actual full date in it that we're going to pass through, which is in the tuple of the index of 0 in our day tuple. We created that is episode 2. Essentially, the dated day contains a number like 0-1 or 1-2 or 3-1, whereas the dated date contains the full date object.

~~~
# rubycast_calendar/app/views/calendars/_day.html.erb
<div data-day="<%= day[0].strftime("%d") %>" 
     data-date="<%= day[0] %>" class="day <%= day[1] %>">
</div>
~~~

Now if we refresh our calendar, and we inspect one of these dates, we can see that it has the dated day of 19 and the dated `date` of 2015-05-19. Now, we can pass that through and use it in our JavaScript. We're going to save off that date value by accessing this keyword in JavaScript. We're going to wrap it in jQuery which gives us access to the data method which simply pulls out the attribute for data and the matching key. We want 'date' so we're going to set that value to the variable `date`. Then we're going to pass through here in parameters. That's going to need a key because it's a hash value. We'll call it date. `Date` is going to equal the date value. If we refresh this again and we click the 19th, it now takes us to our `new` event, View again, but this time the date has been passed through in the parameters.

~~~
# rubycast_calendar/app/assets/javascripts/calendar.js.coffee

$(document).ready ->
  $(".day.today, .day.future").on "click", ->
    date = $(this).data("date")
    window.location.href="/events/new?date=#{date}"
~~~

In order to persist that value on into the model we need to put it into our form. We're going to do that with a hidden field and we're going to name it `occurs_on`. We're going to give it a value. The value is going to come from the `params` hash where the key is `date`.

~~~
#rubycast_calendar/app/views/events/new.html.erb

<h1>New event</h1>

<%= form_for @event do |f| %>
  <%= f.text_field :name %>
  <%= f.hidden_field :ocurrs_on, value: params[:date] %>
  <%= f.submit %>
<%- end %>
~~~

If we inspect this element and we look underneath this we can see that there is another input. It's hidden and it has the date field that we want. The name of the element is Event and then `[occurs on]` because that's a Rails convention. The model is first and then [the attribute]. Then the id is another Rails convention for passing round form data on objects. That is going to be `events_occurs_on`. Now, I created a migration to add the occurs_on field to our Events table. Then I'll migrate my database.

~~~
class AddOcurrsOnToEvents < ActiveRecord::Migration
  def change
    add_column :events, :ocurrs_on, :date
  end
end
~~~

The next thing we have to do is in the controller add this to the secure params hash so it doesn't get filtered out. Now, let's test it out again. This time we're going to click on the 20th and, we're going to `create a date`, just call it `Future date`. Create event. Takes us back to the calendar. Now let's go into the Rails console and see what our data looks like. Okay, so if we inspect the very last element created, we can see that it has an `occurs_on` of 2015-5-20 which was the date that we clicked, so it worked. It passed from the JavaScript layer into the controller layer into the model layer and was saved off into the database.

~~~
class EventsController < ApplicationController
  def new
    @event = Event.new
  end

  def create
    @event = Event.new(event_params)
    if @event.save
      redirect_to calendar_path
    else
      render :new
    end
  end

private

  def event_params
    params.require(:event).permit(:name, :ocurrs_on)
  end
end
~~~

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/007.png "calendario")

The last thing is, we need to display this event onto our calendar underneath the correct date. That's going to happen inside of the stay partial. All we have to do now is render out the partial located at `events/event`. We need to give it some information, some data. We're going to do that with the `collection` key word. What we're going to do is, we're going to pass in `Event.for_date`. That's going to take a `date` and the date is going to be the date that we passed through so it's the first part of our tuple.

~~~
# rubycast_calendar/app/views/calendars/_day.html.erb

<div data-day="<%= day[0].strftime("%d") %>" 
     data-date="<%= day[0] %>" class="day <%= day[1] %>">
     <%= render partial: 'events/event', collection: Event.for_date(day[0]) %>
</div>
~~~

Now, we're going to have to create that `scope` inside of our model. When I refresh the page you can see that we do indeed have our event displaying on the calendar for the 20th which is what we selected.

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/008.png "calendario")

![Episode 005](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/005/images/009.png "calendario")


That's all the time that I have this week for RubyCasts. Next week we're gong to start a new series aimed a little bit more beginner content, by request, for those of you who are graduating or in Dev Bootcamps and haven't spent too much time in Rubyland. I will see you next Tuesday.