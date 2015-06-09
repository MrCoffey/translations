Removing N+ SQL Queries for Speed
====================


We have the ability to add events and show them on the correct date. However, if we look at the SQL query that is generated, we are actually querying the database 42 times (one for every date on the calendar). We want to limit the amount of times we touch the database to make our application more efficient. So how do we fix this?

![Episode 006](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/006/images/002.png "calendar_sql_query")

First, we'll start with the partial where this is occurring: `app/views/calendars/_day.html.erb`. This is rendered 42 times, meaning we have an `Active Record` scope that gets called each time. Instead, let's pass in some data. We are currently passing in this `day` variable which has two values: (1) the date the day partial should render and (2) the collection of CSS classes that are rendered with the partial. We'll be adding a third one, which are the events we're going to render because they belong to a specific date.

```ruby
<div data-day="<%= day[0].strftime("%d") %>" 
  data-date="<%= day[0] %>" class="day <%= day[1] %>">
  <%= render partial: 'events/event', collection: day[2] %>
</div>
```

Open `lib/calendar.rb`. In the `initialize` method, we want to optionally pass in events. However, we already have an optional parameter of `date` and we can't have two. Forunately, Ruby 2 gives us a way around this. We can use keyword arguments. We'll create a keyword argument called `events` that defaults to an empty array and a keyword argument called `date` that defaults to `Date.today`. We'll need to create an instance variable and set it to the `events` array.

```ruby
def initialize(events: [], date: Date.today)
  @events = events
  @date = date
end
```

We'll have to persist those events that we're passing in. Create a private method called `events_for` that takes a date as a parameter. We're going to `select` over the `events` instance variable and check to see if it occurs on the date that was passed in and, if it does, return it as a new array of events. The events of this particular day will be put in to the tuple of the `to_a` method.

```ruby
def to_a
  CalendarWeeks.new(@date).to_a.map do |week|
    week.map do |date|
      [date, DayStyles.new(date).to_s, events_for(date)]
    end
  end
end

private

def events_for(date)
  @events.select { |e| e.occurs_on == date }
end
```


Inside `app/controllers/calendars_controller.rb`, create a private method called `events`. We want to get all of the events that are between two dates. So pass in the first and last calendar dates.

```ruby
private

def events
  @events ||= Event.where("occurs_on BETWEEN ? AND ?", first_calendar_date, last_calendar_date)
end
```

In a previous lesson, we have already defined `first_calendar_date` and `last_calendar_date` in our calendar object. However, we're not going to grab those methods since we already have access to Active Support, so we'll just define new private methods here.

```ruby
def first_calendar_date
  Date.today.beginning_of_month.beginning_of_week(:sunday)
end

def first_calendar_date
  Date.today.end_of_month.end_of_week(:sunday)
end
```

In our `show` method, we're going to need to pass the events of the Active Record layer into the calendar. Call in the keyword parameter of `events` and set it to the private method that we just created.

```ruby
def show
  @calendar = Calendar.new(events: events).to_a
end
```

Restart the rails server and refresh the page. This should now result in a single SQL query instead of the 42 we started out with. Going to the logs, we can see we have a lot of events and day partials being rendered, but there is only one call being made to the database.

![Episode 006](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/006/images/003.png "calendar_sql_query")

That completes the Calendar application and this series. Next week, we'll start a new app and I hope to see you then!