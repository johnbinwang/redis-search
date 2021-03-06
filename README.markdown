# Redis-Search

High performance real-time search (Support Chinese), index in Redis for Rails application

[中文介绍和使用说明](https://github.com/huacnlee/redis-search/wiki/Usage-in-Chinese)

## Status

[![CI Status](https://secure.travis-ci.org/huacnlee/redis-search.png)](http://travis-ci.org/huacnlee/redis-search)

## Features

* Real-time search
* High performance
* Segment words search and prefix match search
* Support ActiveRecord and Mongoid
* Sort results by one field
* Homophone search, pinyin search
* Conditions support

## Requirements

* Redis 2.2+

## Install

in Rails application Gemfile

    gem 'redis','>= 2.1.1'
    gem 'chinese_pinyin', '0.4.1'
    gem 'rmmseg-cpp-huacnlee', '0.2.9'
    gem 'redis-namespace','~> 1.1.0'
    gem 'redis-search', '0.6.3'

install bundlers

    $ bundle install

## Configure

create file in: config/initializers/redis_search.rb

    require "redis"
    require "redis-namespace"
    require "redis-search"
    # don't forget change namespace
    redis = Redis.new(:host => "127.0.0.1",:port => "6379")
    # We suggest you use a special db in Redis, when you need to clear all data, you can use flushdb command to clear them.
    redis.select(3)
    # Give a special namespace as prefix for Redis key, when your have more than one project used redis-search, this config will make them work fine.
    redis = Redis::Namespace.new("your_app_name:redis_search", :redis => redis)
    Redis::Search.configure do |config|
      config.redis = redis
      config.complete_max_length = 100
      config.pinyin_match = true
    end

## Usage

bind Redis::Search callback event, it will to rebuild search indexes when data create or update.

    class Post
      include Mongoid::Document
      include Redis::Search
  
      field :title
      field :body
      field :hits
  
      belongs_to :user
      belongs_to :category
  
      redis_search_index(:title_field => :title,
                         :score_field => :hits,
                         :condition_fields => [:user_id, :category_id],
                         :ext_fields => [:category_name])
  
      def category_name
        self.category.name
      end
    end
    
    class User
      include Mongoid::Document
      include Redis::Search
      
      field :name
      field :tagline
      field :email
      field :followers_count
      
      redis_search_index(:title_field => :name,
                         :prefix_index_enable => true,
                         :score_field => :followers_count,
                         :ext_fields => [:email,:tagline])
    end

    class SearchController < ApplicationController
      # GET /searchs?q=title
      def index
        Redis::Search.query("Post", params[:q], :conditions => {:user_id => 12})
      end
      
      # GET /search_users?q=j
      def search_users
        Redis::Search.complete("Post", params[:q], :conditions => {:user_id => 12, :category_id => 4})
      end
    end

## Index data to Redis

If you are first install it in you old project, or your Redis cache lose, you can use this command to rebuild indexes.

    $ rake redis_search:index

## Documentation

see [Rdoc.info redis-search](http://rubydoc.info/gems/redis-search)

## Benchmark test

You can run the rake command (see Rakefile) to make test.
There is my performance test result.

* [https://gist.github.com/1150933](https://gist.github.com/1150933)
    
## Demo

You can try the search feature in [`zheye.org`](http://zheye.org) | [`ruby-china`](http://ruby-china.org) | [`shu.im`](http://shu.im)
