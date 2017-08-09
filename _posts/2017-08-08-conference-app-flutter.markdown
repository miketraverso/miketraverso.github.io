---
layout: post
title:  "One codebase, two apps and 1278 lines of code"
date:   2017-08-08 16:33:33
description: 'Flutter is a new Google SDK that allows you to create one codebase in Dart and create native apps for iOS and Android. I just used it to create two mobile apps with less than 1200 lines of code.'
tags: android ios programming development dart flutter
thumbnail: /blog/images/posts/thumbnails/flutter-conf-app.png
---

I just used Dart+Flutter to create two mobile apps. I did it in less than 1200 lines of code. Did I mention they're native apps and NOT webapps?

## __Windy City DevCon & DevFest Florida__

DevFest Florida is one of the many DevFests that GDG groups around the country are putting on this fall. GDG Sun Coast, the group I help organize is one of the groups that are involved in organizing the event in Florida. This year we'll be at Disney World on November 11th and at the moment you can score a ticket for $80 if you follow our [twitter feed](https://twitter.com/devfestflorida).

We have a decent [website](https://devfestflorida.org) but folks like to have an app to check the schedule. One of the best apps I've seen is the [Windy City DevCon Android app](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=4&cad=rja&uact=8&ved=0ahUKEwipy_LMjMbVAhXE4yYKHZ5BBUkQFgg5MAM&url=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Dcom.gdgchicagowest.windycitydevcon%26hl%3Den&usg=AFQjCNGIiGrOPsNnUVd42azPLthDxMOO5A). It's an [open source project](https://github.com/rharter/windy-city-devcon-android) written by [Ryan Harter](https://twitter.com/rharter). It's slick in that it clearly shows all the sessions for a specific time block. It shows speaker details. It allows users to favorite their sessions to keep track of what they're interested in. It allows attendees to rate the sessions. It's even written in Kotlin. So what's not to like? Well the ONLY problem with this app is that it's only available for Android.

Cut to I/O 2017 this year. [Emily Fortuna](https://twitter.com/bouncingsheep) and Emily Shack demo'd Flutter and using it and Dart to write one code base that ran natively on iOS and Android. My mind was blown. I wanted to do something in Dart and I figured I could use the great design of the Windy City DevCon app and duplicate it with Flutter.

Here's a preview of what we built so that you'll keep reading. (Since I have no readers... Hi Mom!)

![Side by side preview of the DevFest Florida conference app built with Flutter]({{ site.baseurl }}/images/flutter-conf-app/ios-android.png){:
.img-responsive .center-block }

Don't mind the slow mode in the upper right hand corner. That's Flutter telling you that you're on a debug version, which is what you'll see when you're on an emulator, or when debugging on an actual device. You can see there's a different color of text on each session title. The header is formatted the way it would be natively on each platform too. The cool part that you don't see is that the app is pulling data from Firebase. Let's take a look at the Firebase database structure.

> ![Overall Firebase structure]({{ site.baseurl }}/images/flutter-conf-app/firebase-overall.png){: .img-responsive
.center-block }
>
> An overall look at the Firebase database.

> ![Schedule data structure]({{ site.baseurl }}/images/flutter-conf-app/firebase-schedule.png){: .img-responsive
.center-block }
>
> Schedule data

The overall schedule is specified with dates and an array of timeslots. Each timteslot has a start and an end time. We could have a formatted time stamp or even an epoch but we're not trying to kill a fly with a tank. Just a string will  do. Each sessions is an array holding session ids that tie to a session.


> ![Sessions]({{ site.baseurl }}/images/flutter-conf-app/firebase-sessions.png){: .img-responsive
.center-block }
>
> Session data

Each session has some basic info. An id, title, track, description, room and language. Additionally there is an array of speakers and those ids tie the session to 1 or more speakers.

> ![Speaker data]({{ site.baseurl }}/images/flutter-conf-app/firebase-speaker.png){: .img-responsive
.center-block }
>
> Speaker data

Speaker objects have the usual information. Note the social array. Currently the app only displays the twitter information. It can of course handle more but I'm keeping with the original Windy City DevCon app design

## __Where to find the code__ ##

The code can be found over at [http://github.com/miketraverso/devfest_florida_app](http://github.com/miketraverso/devfest_florida_app). You can download or fork the repo.

## __Project directories__ ##

All the Flutter code can be found in the __lib__ directory. The iOS workspace is in the __ios__ folder and the Android project is in the __android__ folder. I won't get into the specifics of getting your projects setup and building in Xcode and Android Studio respectively. Two useful links though are [Preparing an iOS release](https://flutter.io/ios-release/) and [Preparing an Android release](https://flutter.io/android-release/).

## __DevFest Florida conference app code__ ##
In __`main.dart`__ we have some defaults that we've set up. I've tried to make it easy to configure the code to meet your own needs. Colors, app title, padding, link to a survey, and location data for displaying a static Google map with address and phone number of your location.

<pre><code class="dart">@override
Widget build(BuildContext context) {
  var routes = <String, WidgetBuilder>{
    ScheduleHomeWidget.routeName: (BuildContext context) =>
        new ScheduleHomeWidget(),
    SpeakerListWidget.routeName: (BuildContext context) =>
        new SpeakerListWidget(),
    SpeakerDetailsWidget.routeName: (BuildContext context) =>
        new SpeakerDetailsWidget(),
    SessionDetailsWidget.routeName: (BuildContext context) =>
      new SessionDetailsWidget(),
    LocationWidget.routeName: (BuildContext context) =>
        new LocationWidget()
  };

  return new MaterialApp(
    title: kAppTitle,
    theme: kTheme.copyWith(platform: &#95;platform),
    routes: routes,
    home: new ScheduleHomeWidget(),
  );
}
</code></pre>

We define routes to allow our `MaterialApp` to switch between screens, or in Dart terms, widgets. This overriding build method exists in the `ConfAppState` class. It's responsible for creating the app and setting the home widget that's first seen by the user when the app loads up.

The Firebase package has some UI components, like `FirebaseList` and `FirebaseAnimatedList`. I should have used this to avoid having non-visible components from being generated before they're actually visible. Nothing that can't be fixed. Our schedule_main.dart holds ScheduleHomeWidget and the ConfAppHomeState. In ConfAppHomeState, we have some containers to handle timeslots, schedules and a hash map for speaker ids and speakers. We load from Firebase using the async package so that we're not blocking on the main thread, which with Dart is the only thread.

<pre><code class="dart">LinkedHashMap&lt;String, Session&gt; sessions = kSessions;
LinkedHashMap&lt;String, Speaker&gt; speakers = kSpeakers;
List&lt;TimeSlot&gt; timeSlots = &lt;TimeSlot&gt;[];
List&lt;Schedule&gt; schedules = &lt;Schedule&gt;[];

@override
void initState() {
  super.initState();
  kSpeakers.clear();
  kSessions.clear();
  kTimeSlots.clear();
  loadData();
}

Future loadData() async {
  await loadDataFromFireBase();
}

Future loadDataFromFireBase() async {
  final reference = FirebaseDatabase.instance.reference().child('2017');
  reference.onChildAdded.forEach((e) {
    fireb.DataSnapshot d = e.snapshot;
    if (d.key == 'sessions') {
      LinkedHashMap hashMap = e.snapshot.value;
      hashMap.forEach((key, value) {
        Session session = new Session.loadFromFireBase(key, value);
        sessions.putIfAbsent(session.id, () => session);
      });
      setState(() {
        kSessions = sessions;
        &#95;setStoredFavorites();
      });
    } else if (d.key == 'speakers') {
      LinkedHashMap hashMap = e.snapshot.value;
      hashMap.forEach((key, value) {
        Speaker speaker = new Speaker.loadFromFireBase(key, value);
        speakers.putIfAbsent(speaker.id, () => speaker);
      });
      setState(() {
        kSpeakers = speakers;
      });
    } else if (d.key == 'schedule') {
      for (LinkedHashMap map in d.value) {
        Schedule schedule = new Schedule.loadFromFireBase(d.key, map);
        kSchedules.add(schedule);
        kSchedules.first.timeSlots.forEach((timeSlot) {
          timeSlots.insert(timeSlots.length, timeSlot);
        });
        setState(() {
          kTimeSlots = timeSlots;
        });
      }
    }
  });
}</code></pre>

Each time we process a section of data we call setState() and set our local collections to our app wide collections. I'm new to Dart & Flutter so decided to have an app wide memory cache because I wasn't sure how to handle storing data on both an iOS device and an Android device.

<pre><code class="dart">
@override
Widget build(BuildContext context) {
  List&lt;Widget&gt; timeSlotWidgets = &lt;Widget&gt;[];
  kTimeSlots.forEach((timeSlot) {
    timeSlotWidgets.add(buildScheduledSession(context, timeSlot));
  });

  if (timeSlotWidgets.length == 0) {
    return new Scaffold(
      appBar: new AppBar(
          title: new Text(
            kAppTitle,
            style: new TextStyle(color: Colors.white, fontSize: 24.0),
          )),
      drawer: new ConfAppDrawer(),
      body: const Center(
        child: const CupertinoActivityIndicator(),
      ),
    );
  } else {
    return new Scaffold(
      appBar: new AppBar(
          title: new Text(
            kAppTitle,
            style: new TextStyle(color: Colors.white, fontSize: 24.0),
          )),
      drawer: new ConfAppDrawer(),
      body: new Scrollbar(
        child: new ListView(
          padding: new EdgeInsets.symmetric(vertical: 8.0),
          children: kTimeSlots.length > 0 ? timeSlotWidgets : null,
        ),
      ),
    );
  }
}
</code></pre>

The `ConfAppHomeState` build method, where the widget is created, checks to see if there have been any time slots loaded from Firebase. If there are time slots loaded then we return back a new Scaffold with our app bar and our `ConfAppDrawer` but also a scrolling `ListView` widget where the children are widgets we built to represent each time slot and all of that time slot's sessions. If no time slots have been loaded we return a new Scaffold but with a centered Cupertino (read: iOS style spinner) to tell the user we're still loading.

Finally, when we tap on one of the sessions we want to be able to see the session details. Unlike Android & iOS, there aren't anonymous inner classes to override an onClick event that launches a new Activity/Fragment through an intent OR an outlet that segues to a different part of the storyboard. No, in Dart we have routes. We briefly mentioned them earlier when we defined them for our app.

<pre><code class="dart">Widget buildSessionCard(Session session) {
  String speakerString = getSpeakerNames(session);
  Widget card = new Card(
    child: new GestureDetector(
      onTap: () {
        kSelectedSession = session;
        kSelectedTimeslot = timeSlot;
        log('card tapped');
        Timeline.instantSync('Start Transition', arguments: &lt;String, String&gt;  {
          'from': '/',
          'to': SessionDetailsWidget.routeName
        });
        Navigator.pushNamed(context, SessionDetailsWidget.routeName);
      },
      child: new Container( // ...
</code></pre>

When we build out our sessions we want the user to tap them and be taken to the `SessionDetailsWidget`. In the snippet above you can see we do that by making the child of the `Card` widget a `GestureDetector`. In the `onTap` field we have our handler that tells the `Navigator` to push the new widget into view. Then our actual UI is specified in the child of the `GestureDetector`. Pretty neat.

I'm sure there is a way to pass along data to a new widget but being new to Dart & Flutter I haven't found it yet. That's why you see the setting of `kSelectedSession` and `kSelectedTimeslot` in the `onTap` body. Those fields are being referenced from __`main.dart`__ and help the app keep track of state.

## __Wrapping up__ ##
Anyways that's a brief introduction to Flutter with the [DevFest Florida] app. You should definitely check out Dart & Flutter for your app development needs! Coding with Flutter the past week has brought back the joy of mobile development for me. Way back when, doing mobile development consisted of either BREW for a variety of handsets or you wrote J2ME for devices like the Blackberry blueberry. J2ME at the time didn't have a way to process XML or SOAP. You had to use an external lib like kXML or kSOAP. As frustrating as that was it was less frustrating than what Xcode does to you or even spinning up an Android emulator. I encourage all of you to give it a shot by checking out the following resources:

## __TL;DR__ ##

With libraries that extend Flutter like the Firebase packages allow you to do some pretty cool things. Additionally, you're able to use native code on each platform to continue to extend your app and then pass control back and forth. I'm looking forward to using Flutter and Dart in the future because of how easily I was able to write a pretty complicated conference app for two platforms in under a week

As always, I hope this helps. If you've got any questions reach out to me on [Twitter](https://twitter.com/traversoft), [Facebook](https://facebook.com/traversoft) or in the comments below.

## __Resources__ ##

### __CodeLabs__ ###
+ [Intro to Dart for Java Developers](https://codelabs.developers.google.com/codelabs/from-java-to-dart/index.html?index=..%2F..%2Findex#0)
+ [Building Beautiful UIs with Flutter](https://codelabs.developers.google.com/codelabs/flutter/index.html?index=..%2F..%2Findex)
+ [Firebase for Flutter](https://codelabs.developers.google.com/codelabs/flutter-firebase/index.html?index=..%2F..%2Findex#0  )

### __Documentation__ ###
+ [Dart](https://www.dartlang.org/tools/dartpad)
+ [Dartpad - Online Dart IDE](https://dartpad.dartlang.org)
+ [Flutter](https://flutter.io)
