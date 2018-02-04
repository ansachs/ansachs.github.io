---
layout: post
title:  "Tournapets"
date:   2017-12-24 12:00:00 -0600
categories: projects
image: "/assets/images/tournapets-entry.png"
---
**Motivation**

Many people love their pets and pet social media is incredibly popular: some Instagram pet feeds have millions of subscribers. March madness is incredibly popular tournament that has some crossover with non basketball fans because of the bracket aspect and competition in predicting the winners. 

**What is Tournapets**

Tournapets allows dog lover and dog owners to view and participate in weekly tournaments based around their pet. There will be a weekly challenge that will garner submissions of images from pet owners and allows site members to vote on the best entries. After the week long preliminary round, Eight dogs have risen to the top and can participate in a weekend bracket tournament. The Tournament consists of three rounds, each will have a separate, but related, challenge. 

**Technologies**

front-end: tournament and preliminary built with React-on-Rails and Webpack, other views are built in Rails
back-end: Rails with PostgreSQL 10 table database to track pets, owners, media submission, voting, and tournaments

**Implementation**

React preliminary:

![markdown-img]({{ "/assets/images/tournapets-preliminary.png" | absolute_url}})

React tournament:

![markdown-img]({{ "/assets/images/tournapets-tournament.png" | absolute_url}})

user profile:

![markdown-img]({{ "/assets/images/tournapets-userprofile.png" | absolute_url}})

post comments on pets

![markdown-img]({{ "/assets/images/tournapets-comment.png" | absolute_url}})

A hot-or-not style display built in react allows people to vote in preliminary:

{% highlight javascript %}
...
sendResult(pts) {
  // console.log(JSON.stringify({entry: this.state.links[0].entry_id, vote: pts}))
  fetch('/vote_reg', {
    method: 'POST',
    credentials: 'include',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-TOKEN': this.state.token
    },
    body: JSON.stringify({entry: this.state.links[0].entry_id, vote: pts})
}).then((response) => {
  console.log(response);
})

}

render () {
    console.log(this.props)
    let setImage;
    if (this.state.links.length > 0) {
      setImage = this.state.links[0].link;
    }
    return (

      <div className="container" id="hotornot">
         <div className="row justify-content-lg-center">
          <div className="col-sm-6" >
            <div className="card" style={border: 'none'}>
              <ImageDisplay image={setImage}/>
              <div className="rating row card-body justify-content-center">
                <div id="soso" className="col-sm-2">
                  <img onClick={(e)=> {this.handleButton(e, 0)}} src={soso} />
                  <div id="soso_text" className="tooltip">meh</div>
                </div>
                <div id="okok" className="col-sm-2">
                  <img  onClick={(e)=> {this.handleButton(e, 3)}}  src={okok} />
                  <div id="okok_text" className="tooltip">It's OK</div>
                </div>
                <div id="great" className="col-sm-2">
                  <img onClick={(e)=> {this.handleButton(e, 5)}}  src={great} />
                  <div id="great_text" className="tooltip">5 out of 5</div>
                </div>
              </div>
            </div>
          </div>
          <div className="col-sm-4" id="scores">
             <TableDisplay tournament_id={this.props.tournament_id} />
          </div>
      </div>
{% endhighlight %}

Tournament view is built in React-on-Rails

{% highlight javascript %}

createRound() {
    ...
    var output = Object.keys(this.state.data).map((round, round_num, array) => {
        let style;
        let tag;
        if (round_num === this.props.seeded - 2) {
            style = this.state.current;
            tag = <div id="round-tag">Current Round</div>
          } else {
            tag = <div>&nbsp;</div>
          }

        const output = ( 
          <ul className='round' style={style} key={round}> 
            {tag}
            {this.createMatch(round, round_num)}
            
          </ul>
          );

        return output;
    });

    const wrapped = (
      <div className='bracket'>
        {output}
      </div>
      )
    // console.log(test)
    return wrapped
  }


render () {
    // console.log(this.props)
    const test = this.createRound();
    let popup;
    let notice;
    if (this.state.notice) {
      notice = <h3 id="notice">{this.state.notice}</h3>
      setTimeout(()=>{this.setState({notice: null})}, 5000)
    }
    if (this.state.popup === true) {
      popup = <MatchDisplay popup_state={()=>{this.setState({popup: !this.state.popup})}} popup_data={this.state.popup_data} auth_token={this.state.token} set_notice={(message)=>this.setState({notice: message})}/>
    }

    let title = (<h2 id="title"> {this.props.theme} </h2>)
    return (
        <div>
          {notice}
          {title}
          {test}
          {popup}
        </div>
    );
  }
}

bracket.propTypes = {
  game: PropTypes.object,
  auth: PropTypes.string,
  seeded: PropTypes.number,
  winner: PropTypes.number,
  theme: PropTypes.string
};
}

{% endhighlight %}


Data is passed to React view using special tags. Used fetch to send data from React front end

{% highlight ruby %}

<div class="jumbotron" style="">
  <%= react_component('image_scroller', props = {images: @media, auth: form_authenticity_token, tournament_id: @tournament.id}, prerender: false) %>
<div id="offcolor" style="">

{% endhighlight %}

[Github]: https://github.com/itisjohnday/delta-final-project
[Tournapets]: https://tournapets.herokuapp.com
