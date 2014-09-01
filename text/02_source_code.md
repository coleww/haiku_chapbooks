 Source Code for Haiku eBooks
----------------

<br />

************

<br />

    # haiku_ebooks/Gemfile

    source "https://rubygems.org"

    gem "twitter", "~> 4.8.1"
    gem "tweetstream", "~>2.5.0"

<br />

************

<br />

    # haiku_ebooks/Procfile

    haiku: ruby lib/Haiku.rb

<br />


<br />
<br />
<br />
<br />

************

<br />

    # haiku_ebooks/lib/syllable_dictionary.rb

    # syllable_dictionary.txt is based on the CMU pronuniciation dictionary and looks like this:
    # CAT, 1
    # CATHY, 2
    # etc. (Sorry about that, but we can't show files that are this big right now.)

    dict_path = File.expand_path("../syllable_dictionary.txt", __FILE__)
    dict = File.readlines(dict_path, "r").join.split("\n")

    SYLLABLE_DICTIONARY = Hash.new
    dict.each do |entry|
      word, syllables = entry.strip.split(",")
      SYLLABLE_DICTIONARY[word] = syllables.to_i
    end

<br />

************

<br />

    # haiku_ebooks/lib/web_scraper.rb

    require "twitter"
    require "tweetstream"

    Twitter.configure do |config|
      config.consumer_key       = ENV['TWITTER_CONSUMER_KEY']
      config.consumer_secret    = ENV['TWITTER_CONSUMER_SECRET']
      config.oauth_token        = ENV['TWITTER_OAUTH_TOKEN']
      config.oauth_token_secret = ENV['TWITTER_OAUTH_SECRET']
    end


    TweetStream.configure do |config|
      config.consumer_key       = ENV['TWITTER_CONSUMER_KEY']
      config.consumer_secret    = ENV['TWITTER_CONSUMER_SECRET']
      config.oauth_token        = ENV['TWITTER_OAUTH_TOKEN']
      config.oauth_token_secret = ENV['TWITTER_OAUTH_SECRET']
      config.auth_method        = :oauth
    end

<br />

************

<br />

    # haiku_ebooks/lib/string.rb

    require_relative 'syllable_dictionary.rb'

    class String
      BAD_WORDS = %w(a about ain't aint all an and anything are as at be because
        been but can't do don't dont even feel feels for from gave get getting go
        goes going gonna gonna gotta has have he her his how i i'll i'm if im in
        is isn't isnt it it's its just keep lets like looks make makes me might
        must my no not of on or please really say see she should so some stay such
        take tell that that's thats the their then there there's theres to u up
        used want wants were what whats when where who whole will with won't wont
        would yea yeah you)

      def syllable_count
        self.split(" ").inject(0) do |sum, word|
          sum + SYLLABLE_DICTIONARY[word.upcase]
        end
        rescue TypeError
          return nil
      end



      def haiku?
        return false if self.syllable_count != 17
        words = self.split(" ")
        false_or_words = count_to(5, words)
        false_or_words = count_to(7, false_or_words) if false_or_words
        !!false_or_words
      end

      def count_to(num, words)
        count = 0
        until count == num
          last = words.shift
          count += last.syllable_count
          return false if count > num
        end
        return false if BAD_WORDS.include?(last.downcase)
        words
      end

      def haikuify
        words = self.split(" ")
        haiku = []
        count = 0
        until count == 5
          word = words.shift
          count += word.syllable_count
          haiku << word
        end
        haiku << "/"
        until count == 12
          word = words.shift
          count += word.syllable_count
          haiku << word
        end
        haiku << "/"
        haiku << words
        haiku.join(" ")
      end
    end


************

<br />



    # haiku_ebooks/lib/Haiku.rb

    require_relative 'string.rb'
    require_relative 'syllable_dictionary.rb'
    require_relative 'web_scraper.rb'

    class HaikuFinder
      def self.run_bot
        begin
          TweetStream::Client.new.sample(language: 'en') do |tweet|
            return tweet if tweet.text.haiku?
          end
        rescue
          sleep 300
          retry
        end
      end

      def self.post_tweet(tweet)
        puts "got one!"
        Twitter.retweet(tweet.id)
      end
    end

    loop do
      puts "starting search"
      HaikuFinder.post_tweet(HaikuFinder.run_bot)
      sleep 1200
    end


<br />
<br /><br /><br />

************

<br />

    # haiku_ebooks/spec/string_spec.rb

    require_relative '../lib/string.rb'

    describe String do
      describe ".syllable_count" do
        it "should know it's count" do
          "alpha".syllable_count.should == 2
        end

        it "should know the count of multiple words" do
          "alpha tango bravo nine".syllable_count.should == 7
        end

        it "should handle weird white space" do
          " white   space   ".syllable_count.should == 2
        end

        # maybe I should add poopies to the dictionary?
        # it's kind of absurd that a twitter bot doesn't know about poop jokes
        it "should deal with words that it doesn't know" do
          "poopies".syllable_count.should == nil
        end
      end

      describe ".haiku?" do
        it "should know that 17 is the number" do
          "one two three".haiku?.should == false
        end

        it "should know if a string is a haiku" do
          haiku = "this is a haiku written in order to test that i know haiku"
          haiku.haiku?.should == true
        end

        it "should know that a string is not a haiku" do
          not_haiku = "i am not a haiku though i am made of seven teen syllables"
          not_haiku.haiku?.should == false
        end
      end
    end

<br />

