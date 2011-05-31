!SLIDE

# The Problem #

!SLIDE bullets incremental

## Efficient, Simple Worker Queues ##

* Long-running jobs
* Reduce blocking of web requests

!SLIDE center

<a href="http://redis.io"><img src="redis.png"></a>

!SLIDE bullets incremental

## What is Redis? ##

* Data Structure Server
* Key-Value Access
* Multiple Data Types

!SLIDE bullets incremental

* Strings
* Lists
* Sets
* Sorted Sets
* Hashes

!SLIDE bullets incremental

## ...and more ##

* Atomic Operations
* Pub/Sub
* Transactions

!SLIDE small

## Protocol

    SET key value

!SLIDE small

## Protocol

    LPUSH list item

!SLIDE small

## Ruby

    @@@ ruby
    @redis = Redis.new
    @redis.set("key", "value")
    @redis.lpush("list", "item")

!SLIDE small

## Jobs

    @@@ ruby
    job = {
      id: 1,
      class: "ConfirmationEmailWorker",
      args: ["meyer@paperplanes.de"]
    }

!SLIDE small

## Jobs

    @@@ ruby
    @queue.enqueue(job)

!SLIDE small

## Under the hood

    @@@ ruby
    @redis.lpush("jobs", JSON.dump(job))

!SLIDE smaller

## Under the hood

    LPUSH jobs,
    {"id":1, "class":"ConfirmationEmailWorker", "args":["meyer@paperplanes.de"]}

!SLIDE small

## Workers

    RPOP jobs

!SLIDE small

## Workers in Ruby

    @@@ ruby
    @job = JSON.parse(
      @redis.rpop("jobs")
    )
    Worker.new(@job)

!SLIDE small

## Blocking Workers

    @@@ ruby
    @redis.brpop("jobs")

!SLIDE small

## Mark Job as In-Progress

    @@@ ruby
    @redis.lpush("jobs-in-progress", jobs)

!SLIDE smaller

## [Resque](http://github.com/defunkt/resque)

    @@@ ruby
    class ConfirmationEmailWorker
      @@queue = :confirmation_email

      def self.perform(*args)
      end
    end

!SLIDE smaller

## Resque

    @@@ ruby
    Resque.enqueue(ConfirmationEmailWorker, "meyer@paperplanes.de")

!SLIDE bullets

## Why Redis?

* Simplest Queue Possible
* Atomic Operations
* Counters for statistics
* Visibility
* Fast
