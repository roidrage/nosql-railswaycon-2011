!SLIDE

# The Problem

!SLIDE

## Store and Fetch Activity Feeds Efficiently ##

!SLIDE bullets incremental

## Activity Feed

* User mathias follows user steven
* So do 100000 other users
* Feed of all activities
* Data grows exponentially

!SLIDE bullets incremental

## Solution?

* De-normalize data into streams per user
* Store pre-ordered by time
* Store neighbors close together

!SLIDE center

<a href="http://cassandra.apache.org"><img src="cassandra_logo.png"/></a>

!SLIDE bullets incremental

# What is Cassandra?

* Fault-Tolerant
* Scales up and down
* Hash-like Data Model

!SLIDE bullets incremental

# Data Model

* Rows
* Super Column Families
* Column Families
* Columns

!SLIDE

# Data Model

### It's like a giant, distributed hash ###

!SLIDE

## Super Column Family

    @@@ ruby
    Twotter["mathias"]
           [:Activities]

!SLIDE smaller

## Column Family

    @@@ ruby
    Twotter["mathias"]
           [:Activities]
           [Time.now]

!SLIDE smaller

## Columns

    @@@ ruby
    Twotter["mathias"]
           [:Activities]
           [Time.now] = {"steven" => "had a beer",
                         "type" => "current-status"}

!SLIDE smaller

## Ruby

    @@@ ruby
    require 'cassandra/0.7'
    include SimpleUUID

    cassie = Cassandra.new('Twotter', '127.0.0.1:9160')
    cassie.insert(:Activities, 'mathias', {
      UUID.new => {'steven' => 'had a beer',
                       'type' => 'current-status'}
    })

!SLIDE smallest

## Lots of Columns

    @@@ ruby
    Twotter["mathias"]
           [:Activities] =
             {1.hour.ago  =>
               {"steven" => "had a beer",
                "type" => "current-status"}},
             {2.hours.ago  =>
               {"steven" => "had a beer",
                "type" => "current-status"}},
             {3.hours.ago  =>
               {"steven" => "had a beer",
                "type" => "current-status"}},
             {4.hours.ago  =>
               {"steven" => "had a beer",
                "type" => "current-status"}},

!SLIDE smaller

## Fetch Ranges

    @@@ ruby
    cassie.get(:Activities, 'mathias',
               :start => UUID.new(2.hours.ago),
               :reversed => true)

!SLIDE bullets incremental

## Why Cassandra?

* Data is Presorted by Time
* Scales up on demand
* Locality of Neighboring Data Matters

!SLIDE bullets incremental

## Storage Model

* CommitLog
* MergeTable (in memory)
* SSTable (on disk)
