At the end of last week's episode we created a calendar like you see here before you. 

![Episode 006](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/006/images/001.png "calendario")

This calendar can have events added to it and those events show up on the correct calendar date. But there is a problem here lurking under the hood. If we were to look at the SQL query that gets generated for this we can see that we don't have one SQL query. We in fact have 42 SQL queries, one for every date on that calendar. That's not going to cut it. Today we're going to fix that.

![Episode 006](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/006/images/002.png "calendario")

Let's start with the offending partial which is here in `calendar/views/calendar_days`. This partial gets rendered 42 times. Since it gets rendered 42 times in this particular calendar we have a `active record` scope that gets called 42 times. Rather than that let's expect the data to be passed in. What I mean by that is we are currently passing in this `day` variable, and it's in array, it's actually a tuple. It currently has two values. The first value is the date that the day partial should render. The second value is the collection of CSS classes that should be rendered for the partial. The third one that we’re going to add now is going to be the events that we're going to render in the partial because they belong to a specific date.

```ruby
# ruby-calendar/app/views/calendars/_day.html.erb

<div data-day="<%= day[0].strftime("%d") %>" 
     data-date="<%= day[0] %>" class="day <%= day[1] %>">
     <%= render partial: 'events/event', collection: Event.for_date(day[0]) %>
</div>
```

We need to open up our calendar. We're going to optionally be able to pass in events now. But that presents us with a problem. We already have an optional parameter here date and we can't have two. Fortunately Ruby too gives us a way around this. We can use keyword arguments, create a keyword argument called `events` which defaults to an empty array and a keyword argument called `date` which is going to default to `date.today`. 

Now we're going to have to persist those events that we passed in. We’ll use an instance variable and we're going to set it to the events that we passed in. Here is the tuple that contains the date and the styles, one for each of the dates that are in the array. What we're going to do here is add another value to the tuple and it's going to be a local private method called `events` for which takes a date. 

Now let's create that private method. It’s going to be called `events_for`. It takes a date. We're going to select over the events instance variable. We're going to then check of each these events and it occurs on date. If that date is the same as the date that was passed into the method we will return it as a new array of events. 

```ruby
#ruby-calendar/lib/calendar.rb

class Calendar
    def initialize(events: [], date: Date.today)
        @events = events
        @date = date
    end

    def to_a
        CalendarWeeks.new(@date).to_a.map do |week|
          week.map do |date|
            [date, DayStyles.new(date).to_s, events_for(date)]
          end
        end
    end
  
private 

    def events_for(date)
        @events.select { |e| e.ocurrs_on == date}
    end
end

```

Now the events for this particular day will be put into the tuple. The last thing that we're going to need to change here is inside of our calendar's controller we're going to need to pass the events from the `active record` layer into the calendar. We're going to do that by calling its keyword parameter `events`, and we're going to set it to just a local method. That's private called events. 

I do this because I really don't like local variables strung out through my method. If I have a local variable it's very likely that it could be a local method. I’m going to extract that out. It’s a method now called `events`. This method is going to memorize an array of events to `event.where` and we're going to need to look at the occurs on `date` and get all of the ones where it is between date one and date two which is represented by these two question marks. 

Since we pass two question marks to the SQL query we're going to need to give it two values. The first one is first calendar date, and the second one is last calendar date. If you've been following this series you may notice that we have first calendar date and last calendar date defined deep down in that calendar object. But I'm not going to pull those private local details out of the encapsulation that I had it in just so I can use it here in the controller. 

It doesn't make sense to go from Rails which has `active support` into a Ruby class to get access to `active support` to come back to the Rails. We're just going to create it right here, and it looks like this. We’re going to start off with `date.today`. Then we're going to go to the `beginning_of_month` and the `beginning_of_the_week` starting with Sunday. I'm just going to copy this method because the next method looks a lot like this method with some minor differences. We're going to go to the `last_calendar_day` and we're going to do `end of month` and `end_of_week` starting at Sunday. 

```ruby
require 'calendar'

class CalendarsController < ApplicationController
    def show
        @calendar = Calendar.new(events: events).to_a
    end

private

    def calendar
        @calenndar ||= Event.where("ocurrs_on BETWEEN ? AND ?", first_calendar_date, last_calendar_date)
    end
    
    def first_calendar_date
        Date.today.beginning_of_month.beginning_of_week(:sunday)
    end
    
    def last_calendar_date
        Date.today.beginning_of_month.end_of_week(:sunday)
    end
end
```

Our work here should be complete. We should be able to restart a Rail server, then refresh the page and have a single SQL query instead of 42. Our page has reloaded and we don't notice any change in functionality at, which is good. We go to the logs and we can see that we have a lot of events and a lot of day partials being rendered, but there is only one call to the database which is right here with this events load from events where occurs on between that date and the other date. 

![Episode 006](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/006/images/003.png "calendario")

That wraps it up for the calendar application and this series on Rubycasts. Next week we will start a new app. I hope to see you then. 
