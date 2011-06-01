!SLIDE

# The Problem #

!SLIDE

## Notify Web Frontends of Backend Changes ##

!SLIDE

## [WebSockets](http://www.html5rocks.com/tutorials/websockets/basics/) ##

!SLIDE bullets incremental

## [EventMachine](http://rubyeventmachine.com) ##

* Non-blocking I/O for Ruby
* [em-websocket](https://github.com/igrigorik/em-websocket)
* [em-hiredis](https://github.com/mloughran/em-hiredis)

!SLIDE

## Redis Pub/Sub ##

!SLIDE small

## Sub ##

    SUBSCRIBE notifications

!SLIDE small

## Pub ##

    PUBLISH notifications "Mathias signed up"

!SLIDE small

## Pattern-Matching Subscriptions ##

    PSUBSCRIBE notifications.*

!SLIDE small

## Publish ##

    PUBLISH notifications.new-user "Mathias signed up"

!SLIDE smaller

## WebSocket Server ##

    @@@ ruby
    EventMachine::WebSocket.start(:host => '0.0.0.0', :port => 8081) do |ws|
      ws.onopen do
        SOCKETS << ws
      end

      ws.onclose do
        SOCKETS.delete ws
      end
    end

!SLIDE small

## Subscriber ##

    @@@ ruby
    @subscriber = EM::Hiredis.connect
    @subscriber.subscribe("notifications")

    @subscriber.on(:message) do |channel, data|
      SOCKETS.each {|socket| socket.send(data)}
    end

!SLIDE smaller

## Subscribe on patterns

    @@@ ruby
    SOCKETS = {}
    ws.onmessage do |pattern|
      SOCKETS[pattern] ||= []
      SOCKETS[pattern] << ws
      @subscriber.psubscribe(pattern)
    end

!SLIDE smaller

## Subscriber    

    @@@ ruby
    @subscriber.on(:pmessage) do |pattern, channel, data|
      SOCKETS[pattern].each {|socket| socket.send(data)}
    end

!SLIDE smaller

## Your Web Frontend

    @@@ javascript
    websocket = new WebSocket("ws://localhost:8081");
    websocket.onmessage = function(evt) { onMessage(evt); };
    setTimeout(function() {i
      websocket.send("new-users");
    }, 1000);

!SLIDE smaller

## Your Rails App

    @@@ ruby
    class User
      after_create :notify_subscribers

      def notify_subscribers
        redis.publish("new-users")
      end
    end

!SLIDE

# Demo

!SLIDE websockets-demo center bullets

<ul></ul>

<script>
$(".websockets-demo").bind("showoff:show", function (event) {
  $(this).children('ul').each(function() {
    $(this).children().remove();
    var self = this;
    function onMessage(evt) {
      var li = $(document.createElement('li'));
      li.text(evt.data)
      $(self).append(li);
    }
    websocket = new WebSocket("ws://localhost:8081");
    websocket.onmessage = onMessage;
    websocket.onopen = function(evt) {
      var li = $(document.createElement('li'));
      li.text("Connected to WebSocket server");
      $(self).append(li);
    };
    websocket.onclose = function(evt) {
      var li = $(document.createElement('li'));
      li.text("Can't connect to WebSocket server");
      $(self).append(li);
    };
    setTimeout(function() {websocket.send("new-users");}, 1000);
  });
});
</script>

!SLIDE bullets incremental

# Why Redis?

* Built-in Pub/Sub
* Persistence on top
* Backend/Frontend Glue

!SLIDE bullets incremental

# Redis

* Fast
* Simple
* Swiss Army Knife Database
* Not Distributed (yet)
* Limited by memory (not for long)
