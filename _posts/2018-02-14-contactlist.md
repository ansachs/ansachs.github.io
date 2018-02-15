---
layout: post
title: contactlist
date: 2018-02-14 12:00 -0600
categories: projects
image: "/assets/images/contactlist-overall.png"
summary: | 
  simple React app with a contact list that is sorted by favorite status. Favorite status can be altered by clicking on the star
---

**Motivation**

A project to improve my skills in React and utilize an API call 

**What is Contact List**

A single page React app with contacts that are fetched by an API call and sorted according to favorite status. Clicking on a contact brings up an info page on the person. Contacts can be added and removed from the list by clicking on the star.

**Technologies**

List: ReactJS 
testing: Jest, Enzyme

**Implementation**

A user can select an person on the contact list and view their personal info:

![markdown-img]({{ "/assets/images/contactlist-overall.png" | absolute_url}}) 

By clicking on the star in the right hand corner of personal info a user can change the favorite status of the contact. The star will go from unfilled to filled or vice-versa:

![markdown-img]({{ "/assets/images/contactlist-add_to_favorite.png" | absolute_url}}) 

The personal info can be close by clicking on the contacts link in the left hand conder of the personal info page:

![markdown-img]({{ "/assets/images/contactlist-single_view.png" | absolute_url}}) 

The API had no 'CORS allowed' header and had to be called with a proxy that added the property

{% highlight javascript %}
{% raw %}

static fetchContacts() {
    const proxyUrl = <my proxy>,
      targetUrl = 'https://s3.amazonaws.com/technical-challenge/v3/contacts.json'

    return fetch(proxyUrl + targetUrl)
      .then(response => {return response.json();})
      .catch((err) => {
          console.log(err)
        })
  }
{% endraw %}
{% endhighlight %}

I mocked the API call to reduce calls to the server while developing:

{% highlight javascript %}
fetchMock
    .get(
      proxyUrl + targetUrl,
      data.contactsData
    )
{% endhighlight %}

One component class had some methods that required testing. I utilized Enzyme shallow rendering to test them:

{% highlight javascript %}
{% raw %}

describe('changeStatus', () => {
    it('changes status from favorite to unfavorite', () => {

      domElement.instance().changeStatus();
      const currentPerson = domElement.state().contacts.find((person)=>{
        return person.id == 1;
      })

    expect(currentPerson.isFavorite).toEqual(true)
    })
  })

{% endraw %}
{% endhighlight %}