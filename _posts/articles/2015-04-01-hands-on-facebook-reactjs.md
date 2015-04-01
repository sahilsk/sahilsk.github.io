---
layout: post
title: "Hands on Facebook #reactjs"
modified:
categories: articles
excerpt:
comments: true
share: true
tags: ['reactjs', 'facebook', 'realtime', 'angularjs']
date: 2015-04-01T08:23:21+05:30
---

With the news of facebook launching [react-native](https://code.facebook.com/videos/786462671439502/react-js-conf-2015-keynote-introducing-react-native-/), i couldn't stop myself from learning [ReactJS](https://facebook.github.io/react/).

Earlier than that, i often ask myself this question on a thought of learning new javascript frameworks : **Why do i need to learn one more?**. 

Wel, News of *react-native* pique my interest and forced me to step out of my comfort zone. I fire up the browser with ignited spirit to devour ReactJS. (:

## First Impression

With a background of MVC taught by backbone and angularjs, reactJS was totally new.
ReactJS is __V__ in __MVC__ . I then thought how's it different from angularJS Directive or ember components?

I also got confused by __one-way reactive data flow__. What does it mean? Data often moves from back & forth.
ReactJS uses virtual DOM. So, how is it useful?

Without mulling over it much, i decided to build an RealTime application using it and when it comes to realTime twitter live feed often comes into mind. 

So, i made Realtime status tracking application using ReactJS:  [http://track-tweets.herokuapp.com/](http://track-tweets.herokuapp.com/). It took me only two days ( need not to forget i go to office in day time) to built this application.

![RealtTime App]({{ site.url }}/images/realtimeApp.png)

Feature list:

- Show tweets on page load (render at server side)
- As user scroll down and reach near bottom, more tweets appends from mongoDB powered database (lazy loading)
- As new tweets show up on twitter matching track string, user get realtime notification about it

Please find source code on [github: sahilsk/track-tweets](https://github.com/sahilsk/track-tweets)

![RealtTime App]({{ site.url }}/images/realtimeApp_full.png)

In this post, i want to share my views on facebook ReactJS.

> NOTE: Views are totally personal and has nothing to do with organization, people or girls i might be connected to..


## Small learning Curve

Well, having removed __M__ and __C__ from __MVC__ you're left with only __V__ to learn and that is __ReactJS__. So, learning curve is drastically reduced. In couple of minutes you can get started and rest you can learn as you go. 

There getting started guide incredibly simple. Why incredible? because i didn't trust it and take google help to search more advance ReactJS tutorials. You can save time if you stick with it( though googling won't harm either).

You don't need to learn about providers, service , factory or any more jargons you may have seen in other frameworks.

## Tidy manageable code

In writing realTime application code often go messy. In reactJS components help you organize your code by allowing you to divide big application into smaller component.

For my application here the application components when put together look like: 

**/react/components/tweetApp.react.js**

``` javascript
---
render: function(){

  return (
    <div className="tweetApp_wrapper"  >
      <Tracker track={this.state.track}  />
      <Tweets tweets={this.state.tweets}   />
      <Notification count={this.state.unreadTweets.count}
        showUnreadTweets={this.showUnreadTweets}
          />
      <Loader loading={this.state.loading} />					
    </div>
      
  )
}
---
```

## Cool stuff

ReactJS **Virtual DoM** render the whole DOM in memory first and then push only the diff into the browser one.
It not only make browser rendering fast, but also enable you to render the DOM at server side. Cool isn't it?

In my application when page open up it's ReactJS server side rendered components that you see.
As shown in the snippet below `renderToString` method render component as string which i'm passing to my [expressJS](expressjs.com) `index` view template.

**/server/routes/index.js**

``` javascript
---
var markup = React.renderToString( TweetApp( {"tweets": tweets } ) );
res.render("index",  
  { "layout": '../../server/views/layouts/main', 
    "tweets": JSON.stringify(tweets), 
   "markup": markup  
   } );
---
```
Why is it important? Well, i care for my website reachability. So, i care about it SEO. 
Though google crawler *[interpret](http://stackoverflow.com/questions/2061844/do-googles-crawlers-interpret-javascript-what-if-i-load-a-page-through-ajax)* javascript, you still have to rely on third party services like [prerender.io](https://prerender.io/) or phantomJS to boost your SEO.


## What next?

[Facebok react-native](http://facebook.github.io/react-native/): React Native enables you to build world-class application experiences on native platforms using a consistent developer experience based on JavaScript and React. 

At the time of writing this article react-native supports only iOS.

[Facebook Flux](https://facebook.github.io/flux/docs/overview.html):  application architecture bolstering uni-directional flow in applications.

[Facebook Flow](https://github.com/facebook/flow) : Adds static typing to JavaScript to improve developer productivity and code quality. 
http://flowtype.org/

...and hopefully more in future. Be hopeful please...


## Conclusion

ReactJS, Flux & Flow are more about philosophical expect of development rather than learning few more technical jargons terms. phew...

My new year resolution is to learn more on ReactJS, REACT-Native, flux & flow.



Source code: [sahilsk/track-tweets](https://github.com/sahilsk/track-tweets)
