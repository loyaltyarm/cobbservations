+++
date = "2014-05-06T01:27:26-07:00"
draft = false
title = "Centering GMSMapView on User's Current Location - Google Maps SDK for iOS"
tags = ["Apple", "Development", "iOS 7", "Software"]
categories = ["Development", "iOS 7", "Software"]

+++

I'm working on a personal project that involves the use of the Google Maps SDK for iOS. As I am still picking up various tidbits and skills related to iOS development, I did a bit of learning first in order to get started. [Code School](https://www.codeschool.com/) recently opened up a free channel for learning about [using Google Maps SDK for iOS](https://www.codeschool.com/courses/exploring-google-maps-for-ios), and that has been pretty helpful. According to the [Google Maps SDK documentation](https://developers.google.com/maps/documentation/ios/), there are various ways to create a mapView and assign its camera position, or force a camera update. You can check out the documentation and tutorials [here](https://developers.google.com/maps/documentation/ios/) for yourself. It is a lot of reading and maybe a bit much to try to cover everything. Watching the tutorial from Code School above should get you started. After learning the basics about creating the GMSMapView, I learned from Stack Overflow that if I wanted to get the user's location, I would have to use [key-value observing](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html). The [thread from Stack Overflow is linked here](http://stackoverflow.com/questions/20607358/could-not-get-current-location-using-google-map-api-gmsmapview/20611812#20611812), and details hows to get the value of the GMSMapView myLocation property, if enabled. I modified it a bit, to fit my use case.

To preface a bit I created, as shown in the Code School demonstrations, my `GMSMapView` within the `viewDidLoad` method of my View Controller, in this case, `BGMapVC`. Since the `GMSMapView` is already created and has settings applied, it will be shown when the user selects my scene while using the device. Again, the trick here is to get the value of the myLocation property and set the map's camera to point at it upon the view appearing. You can find how I implemented this below: 

First, I make sure to set the delegate in my header file:


```
@interface BGMapVC : UIViewController <GMSMapViewDelegate, CLLocationManagerDelegate>
```

Then I set a property for my `mapView`:

```
@property (nonatomic, strong) GMSMapView *mapView;
```

Then, where the answer shows to create the observer within the viewDidLoad method, I have done so within the `viewWillAppear` method, so that it is present prior to my interface loading.

```
- (void)viewWillAppear:(BOOL)animated {
    [self.mapView addObserver:self forKeyPath:@"myLocation" options:0 context:nil];
}
```

Then, call the `dealloc` method to remove the observer:

```
- (void)dealloc {
    [self.mapView removeObserver:self forKeyPath:@"myLocation"];
}
```

Lastly, in the `observeValueForKeyPath:keyPath:ofObject:change:context:` method, I create the reference to the GMSMapView's `myLocation` property using a CLLocation object, and then create a target for the GMSMapView to utilize when updating the camera on the map view (CLLocationCoordinate2D). Then, at the end of the method, I force the `mapView` to update. Since the observer is being called from the `viewWillAppear` method, this tells the map view to update the location prior to the view being loaded. Code below:

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if([keyPath isEqualToString:@"myLocation"]) {
        CLLocation *location = [object myLocation];
        //...
        NSLog(@"Location, %@,", location);

        CLLocationCoordinate2D target =
        CLLocationCoordinate2DMake(location.coordinate.latitude, location.coordinate.longitude);

        [self.mapView animateToLocation:target];
        [self.mapView animateToZoom:17];
    }
}
```

At the end, the `animateToLocation:` and `animateToZoom:` calls update the map's view accordingly.

As always, open to feedback, suggestions, corrections, questions, or criticisms. I am trying to learn and I feel the best way to learn is to document problems I researched while learning and get feedback. I am particularly interested in whether or not there are opportunities for better memory management and if this could be done without KVO.
