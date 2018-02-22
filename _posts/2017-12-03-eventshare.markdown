---
layout: post
title:  "Chicago Eventshare"
date:   2017-12-03 12:00:00 -0600
categories: projects
image: "/assets/images/eventshare-main.png"
summary: |
  event-centric page where users can chat in realtime, view a relevant twitter feed, and post video links through the chat interface
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

    youtube_reg = [
      /^https:\/\/www.youtube.com\/embed\/[0-9a-zA-Z_\-]*$/,
      /^https:\/\/youtu.be\/[0-9a-zA-Z_\-]*$/
      ]

    if params['message']['statement'].strip.match?(Regexp.union(youtube_reg))

      if params['message']['statement'].strip.match?(youtube_reg[1])
        embed = params['message']['statement'].strip.match(/^https:\/\/youtu.be\/([0-9a-zA-Z_\-]*$)/)[1]
        params['message']['link'] = "https://www.youtube.com/embed/" + embed 
      else
        params['message']['link'] = params['message']['statement'].strip
      end


      record = MediaLink.find_or_initialize_by(media_params)
      
      if record.new_record?
        record.user_id = current_user.id
        record.media_type = "video"
        record.concert_id = @concert.id
        record.save
        ActionCable.server.broadcast "room_#{@concert.id}", media: render_video(record)
      else
...
{% endhighlight %}

One of the challenges was setting up Action Cable, which runs on the client in javascript. I setup the chat as socket driven from the server to the client and via HTTP post from client to server. In retrospect, I could have used sockets both ways.

{% highlight javascript %}
...
function submitNewMessage(){
  $('#submit-text').submit(
    
    function(e) {
      e.preventDefault();
      const curr_concert = window.location.pathname.match(/concerts\/(\d*)/)[1];
      const url = '/concerts/' + curr_concert + '/shared_experiences';
      const text = $('[data-textarea="message"]').val()
      const token = $('[name="authenticity_token"]').first().val()

      fetch(url, {
        method: 'POST',
        credentials: 'include',
        headers: {
          'Content-Type': 'application/json',
          'X-CSRF-TOKEN': token
        },
        body: JSON.stringify({message: {statement: text}, concert_id: curr_concert})
      }).then((response) => {
        if (response.redirected == true) {
          $('.notice').html("please register or login before chatting")
          $('[data-textarea="message"]').val("")
          setTimeout(function(){document.querySelector('.notice').innerHTML = ""},10000);
        }
      }).catch( err =>{console.log(err)})
      return false;
  });
  ...
}
{% endhighlight %}

<a href="https://github.com/ansachs/eventshare" target="_blank">Gigshare</a>

<a href="https://gigshare.herokuapp.com/" target="_blank">Gigshare live on Heroku</a>
