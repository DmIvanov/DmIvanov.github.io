---
id: 13
title: HealthKit changes observing
date: 2018-10-08T19:34:38-02:00
author: topolog
layout: post
guid: http://dmtopolog.com/?p=13
permalink: /healthkit-changes-observing/
categories:
  - Tech Blog
tags:
  - HealthKit
  - iOS
---
There are plenty of_ **&#8220;Getting started with HealthKit&#8221;**_ tutorials which tells you how to set up the HealthKit integration, request an access to read and write data and make simple queries. That&#8217;s pretty much enough you need to know if you just need to get user&#8217;s sex, height or recent weight measurement. I assume you already know how to deal with all that. But what if you need to monitor and process the changes in user&#8217;s HealthApp? For this purpose HealthKit provides you _HKObserverQuery_.

&nbsp;

#### **Meet HKObserverQuery**

According the documentation _HKObserverQuery_ is

> A long-running query that monitors the HealthKit store and updates your app whenever a matching sample is saved to or deleted from the HealthKit store.

You create the query, attach some handler to it and execute it in your HKHealthStore.

<pre class="lang:swift decode:true">let bodyMassType = HKQuantityType.quantityType(forIdentifier: HKQuantityTypeIdentifier.bodyMass)!
let hkStore = HKHealthStore()

func startObserving() {
	/*
		`bodyMassObserverQuery` should be a property not a local variable,
		so you'll be able to stop the query when necessary
	*/
	bodyMassObserverQuery = HKObserverQuery(
		sampleType: bodyMassType,
		predicate: nil) { [weak self] (query, completion, error) in
			self?.bodyMassObserverQueryTriggered()
	}
	
	hkStore.execute(bodyMassObserverQuery!)
}

func bodyMassObserverQueryTriggered() {
    let bodyMassSampleQuery = HKSampleQuery(
            sampleType: bodyMassType,
            predicate: nil,
            limit: HKObjectQueryNoLimit,
            sortDescriptors: nil,
            resultsHandler: { [weak self] (query, samples, error) in
                self?.bodyMassSampleQueryFinished(samples: samples)
        })
    hkStore.execute(bodyMassSampleQuery!)
}</pre>

As far as _HKObserverQuery_ doesn&#8217;t give you any samples when it&#8217;s callback being triggered you need to run _HKSampleQuery_ to get the actual data to work with.

Although in documentation it&#8217;s said that _HKObserverQuery&#8217;s_ callback should be called when something is changed in the observing data, in fact it can be called more frequently. For instance, if you don&#8217;t process the changes in the background (we will talk about background mode later) the callback will be triggered every time your app is going foreground (if the query is running), regardless of any changes in HealthKit. So in fact you cannot be sure that you have any new data to process when the callback is triggered.

&nbsp;

#### **Finding out changes**

The next task will be to distinguish changed measurements: the added and the removed ones. There can be several approaches.

##### 1. Brute force comparison (no observers)

If we don&#8217;t need to immediately react on every data change in our app, we can ignore using any observers and just every once in a while (for instance when the app launches or goes foreground) retrieve all the measurements from the HealthKit, compare them with what we have locally and distinguish the changes. Every _HKObject_ has it&#8217;s UUID so we can store this identifiers locally with corresponding samples and use it for comparison. (Prior iOS 8 there were no UUID in HKObjects, so there could be quite tricky ways to compare one object to another&#8230; luckily it&#8217;s in the past)

##### 2. Brute force comparison (with observer)

Every time when _HKObserverQuery&#8217;s_ callback is triggered we retrieve all the measurements from the HealthKit and compare them with what we have in our data base (see the previous point).

##### 3. Using predicate

You could notice that in the initialisers of both _HKObserverQuery_ and _HKSampleQuery_ there are predicate as a parameter, and you know that _HKSamples_ have startDate/endDate as properties, so there should be a way to make a filter for the query. So we can persist the date of the last syncing and use it to retrieve just new measurements (which date is bigger than the \`lastRetrievedDate\`)

<pre class="lang:swift decode:true">let currentDate = Date()
let predicate = HKQuery.predicateForSamples(
    withStart: lastRetrievedDate,
    end: currentDate,
    options: .strictEndDate
)
// create and execute the query
lastRetrievedDate = currentDate</pre>

But it won&#8217;t take you much thinking to figure out that it won&#8217;t work for the changes that were made in past (some samples could be added manually with the \`startDate < lastRetrievedDate\`. This approach also doesn&#8217;t consider deleted samples.

##### 4. HKAnchoredObjectQuery

The release of iOS 8 brought us lots of HealthKit improvements and _HKAnchoredObjectQuery_ among them. From documentation:

> An HKAnchoredObjectQuery returns an anchor value that corresponds to the last sample or deleted object received by that query. Subsequent queries can use this anchor to restrict their results to only newer saved or deleted objects.

Simply this query provide us with database version or _snapshot of the changes_ as you can find it in the documentation (the same approach was introduced as _generations_ for Core Data). _HKQueryAnchor_ represents the version in this example. The first year of it&#8217;s existence it was just an \`int\` which made it look even more like version, but not it&#8217;s an entire object.

Here is an API to create an _HKAnchoredObjectQuery_:

<pre class="lang:swift decode:true ">let anchorQuery = HKAnchoredObjectQuery(
    type: bodyMassType,
    predicate: nil,
    anchor: previousAnchor,
    limit: HKObjectQueryNoLimit,
    resultsHandler: {(query, samplesOrNil, deletedObjectsOrNil, newAnchor, errorOrNil) in
        // process the callback result
        previousAnchor = newAnchor
})</pre>

_(anchor parameter is nullable, so it&#8217;s totally OK to pass nil at the beginning)_

You pass an anchor as a parameter and receive a new one associated with this query when the query returns you the result. Using _HKAnchoredObjectQuery_ you can be sure that it returns you only the diff from the last revision (including deleted objects).

_HKQueryAnchor_ conforms to _NSSecureCoding_ protocol which means it can be easily persisted on disk and used after relaunching the app.

<pre class="lang:swift decode:true">func storeAnchor(anchor: HKQueryAnchor?) {
    guard let anchor = anchor else { return }
    do {
        let data = try NSKeyedArchiver.archivedData(withRootObject: anchor, requiringSecureCoding: true)
        UserDefaults.standard.set(data, forKey: kUserDefaultsAnchorKey)
    } catch {
        print("Unable to store new anchor")
    }
}

func retrieveAnchor() -&gt; HKQueryAnchor? {
    guard let data = UserDefaults.standard.data(forKey: kUserDefaultsAnchorKey) else { return nil }
    do {
        return try NSKeyedUnarchiver.unarchivedObject(ofClass: HKQueryAnchor.self, from: data)
    } catch {
        print("Unable to retrieve an anchor")
        return nil
    }
}</pre>

#### 

#### **HKAnchoredObjectQuery needs no help**

Although _HKAnchoredObjectQuery_ can be used instead of _HKSampleQuery_ when being triggered by the _HKObserverQuery_, but the thing is that actually _HKAnchoredObjectQuery_ doesn&#8217;t need any _HKObserverQuery_ to work with &#8211; it can be an observer itself.  
From documentation:

> A query that returns only recent changes to the HealthKit store, including a snapshot of new changes and continuous monitoring as a long-running query.

For this purpose _HKObserverQuery_ has \`updateHandler\` property which has the same closure semantics as \`resultsHandler\`:

<pre class="lang:swift decode:true ">((HKAnchoredObjectQuery, [HKSample]?, [HKDeletedObject]?, HKQueryAnchor?, Error?) -&gt; Void)?</pre>

If you omit setting the `updateHandler` the query just triggers the `resultsHandler` and finishes. But if you set an `updateHandler`, the query will act as long-running task and triggers it every time it has some changes (every time when you enter foreground if you don&#8217;t have background data processing).

And one more quote from the documentation:

> Often, it is more efficient to setup and run a single anchored object query, then to run separate sample and observer queries.