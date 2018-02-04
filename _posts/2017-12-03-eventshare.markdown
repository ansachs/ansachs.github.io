---
layout: post
title:  "Chicago Eventshare"
date:   2017-12-03 12:00:00 -0600
categories: projects
image: "/assets/images/eventshare-main.png"
---

**Motivation**

The current iteration of do312 websites is very good for finding events and gauging their popularity, but it lacks the ability for users to interact around events. Currently the only site available for interacting around events is facebook, but they lack realtime group chat and and a search exclusively for music events. 

**What is Eventshare**

Eventshare allows users to find events on a certain date by accessing the DoStuffMedia API. Users can enter an event-centric page where they can chat in realtime, view a relevant twitter feed, and post video links through the chat interface.

**Technologies**
realtime chat: realtime chat implemented using Action Cable and Redis pub/sub
Twitter feed: displays a slightly delayed feed of relevant tweets (twitter API limits free searches)
video carousel: utilizes JCarousel to display embedded Youtube videos. Videos are added by pasting links into the chat box


**Implementation**

The music selection page utilizes the DoStuffMedia API to show concerts on a certain date:

![markdown-img]({{ "/assets/images/eventshare-calendar.png" | absolute_url}})

The main Eventshare page has a media carousel, chat window, and a twitter stream windo

![markdown-img]({{ "/assets/images/eventshare-main.png" | absolute_url}})

I used Faraday to connect with the API:


{% highlight ruby %}
require 'faraday'
require 'faraday_middleware'


class LoadConcerts
  def initialize
    @connection = self.create_connection
  end

  def create_connection
    conn = Faraday.new(:url => 'http://do312.com/') do |faraday|
        faraday.request  :url_encoded
        faraday.response :json       
        faraday.adapter  Faraday.default_adapter 
      end
  end

  def concert_logic(params)
    if params['date'] == nil
      date_in_time_type = Time.now
      date_zeroed = date_in_time_type.beginning_of_day
    else
      date_in_time_type = params['date'].to_time
      date_zeroed = date_in_time_type.beginning_of_day
    end
    # the purpose of this logic was to reduce API calls
    if Concert.where(concert_date: ((date_zeroed).to_datetime..(date_in_time_type + 24.hours).to_datetime)).exists?
      return Concert.where(concert_date: ((date_zeroed).to_datetime..(date_in_time_type + 24.hours).to_datetime))
    else 
      return format_data(self.get_listings(date_in_time_type.to_datetime))
    end
  end

  def get_listings(date)
      
      format_date = date.strftime('%Y/%m/%d')
      # binding.pry
      response = @connection.get "events/#{format_date}.json"
      return response.body['events']
  end

  def format_data(concert_array)
    concert_list =[]
    concert_array.each do | event |

        event['artists'].each do |artist|
          if Band.where(title: artist['title']) == nil
            band_info = get_band(artist['permalink'])
            save_band(band_info['artist'])
            ...
{% endhighlight %}

I used the controller to handle links and chat text:

{% highlight ruby %}
...
def create
    if current_user == nil
      redirect_to concert_shared_experiences_path, notice: 'must login in to use chat'
    elsif params['message']['statement'].match?(/^https:\/\/www.youtube.com\/embed\/[0-9a-zA-Z_\-]*$/)
    params['message']['link'] = params['message'].delete('statement')
    new_link = MediaLink.new(media_params)
    new_link.user_id = current_user.id
    new_link.media_type = "video"
    new_link.concert_id = @concert.id
    new_link.save
    ActionCable.server.broadcast "room_#{@concert.id}", media: render_video(new_link)
        render body: nil
    else

      message = Message.new(message_params)
      message.user_id = current_user.id
      message.save

      ActionCable.server.broadcast "room_#{@concert.id}", chat: render_message(message)
        render body: nil
    end
  end
...
{% endhighlight %}

[Eventshare]: https://github.com/ansachs/eventshare
