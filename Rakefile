require 'irb'
require 'pathname'
require 'json'
require 'rake/clean'
require 'time'

class Tweet
  attr_reader :tweet_id, :created_at

  def initialize(row)
    @tweet_id   = row.dig('tweet', 'id')
    @created_at = Time.parse(row.dig('tweet', 'created_at'))
    @row        = row
  end
end

class DayPage
  attr_reader :tweets, :filename
  def self.call(tweets)
    new(tweets).call
  end

  def initialize(tweets)
    @tweets = tweets
  end

  def call
    header +
    tweets.map{|tweet| tweet.created_at.year}.uniq.sort.reverse.map do |year|
      [
        "<h2>#{year}</h2>", 
        tweets.select{|t| t.created_at.year == year }.map(&:tweet_id).map{|id| embed(id)}
      ]
    end.flatten.join("\n") +
    footer
  end

  def embed(id)
    str=<<~EMBEDEOF
      <blockquote class="twitter-tweet" data-lang="en">
        <a href="https://twitter.com/sengming/status/#{id}"></a>
      </blockquote>
    EMBEDEOF
  end

  def header
    str=<<~HEADEOF
      <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta name="viewport" content="width=480">
      </head>
      <body>
    HEADEOF
  end

  def footer
    str=<<~FOOTEREOF
        <script>window.twttr = (function(d, s, id) {
          var js, fjs = d.getElementsByTagName(s)[0],
            t = window.twttr || {};
          if (d.getElementById(id)) return t;
          js = d.createElement(s);
          js.id = id;
          js.src = "https://platform.twitter.com/widgets.js";
          fjs.parentNode.insertBefore(js, fjs);

          t._e = [];
          t.ready = function(f) {
            t._e.push(f);
          };

          return t;
        }(document, "script", "twitter-wjs"));</script>
      </body>
    FOOTEREOF
  end
end

CLEAN.include 'daily/tweets-*.html'

task :parse_json do
  tweet_path = Pathname.new('tweet.js')
  tweet_json = tweet_path.read[25..-1]
  tweets     =  JSON
                .parse(tweet_json)
                .map{ |row| Tweet.new(row) }

  mkdir_p 'daily'

  (1..12).each do |month|
    (1..31).each do |day|
      tweets_on_this_day = tweets.select{|tweet| tweet.created_at.month == month && tweet.created_at.day == day}.sort_by(&:created_at)

      if tweets_on_this_day.any?
        tweet_filename = "daily/tweets-#{sprintf '%02d', month}-#{sprintf '%02d', day}.html"

        puts "Writing #{tweet_filename}..."
        open(tweet_filename, 'w') do |f|
          f.write DayPage.(tweets_on_this_day)
        end
      end
    end
  end
end