
# FLATIRON GO

---


## Firebase & Geofire Component

```swift
let geofireRef = FIRDatabase.database().referenceWithPath(FIRReferencePath.treasureLocations)
        let geoFire = GeoFire(firebaseRef: geofireRef)
        let geoQuery = geoFire.queryAtLocation(location, withRadius: 10.0)
```

* We get a hold of our firebase reference.
* Query up using GeoFire using the users current location.
* In our callback, we receive multiple "Treasure" objects" which we parse through.
* Receiving the coordinates of these "Treasure Objects", we create annotations and place them on the map. These annotations are selectable.


* Our firebase database (Firebase):

![http://i.imgur.com/u8UxY6X.png?1](http://i.imgur.com/u8UxY6X.png?1)

---


## Custom Map w/ MapBox

* Utilizing [MapBox](https://www.mapbox.com/ios-sdk/api/3.3.0/) which allows us to easitly create a custom looking map (like the one being used in the app).
* Through [MapBox](https://www.mapbox.com/ios-sdk/api/3.3.0/), we're creating the custom treasure annotation.
* Our map is customized through the various delegate methods available to us.
* Setting up the Map View which is then added tou our `view`

```swift
mapView = MGLMapView(frame: view.bounds,
                                 styleURL: NSURL(string: "mapbox://styles/ianrahman/ciqodpgxe000681nm8xi1u1o9"))
```

---

## AR Component

**1** - Setup a `AVCaptureSession` utilizing the `AVCaptureDevice`.

```swift
private func setupCaptureCameraDevice() {
        let cameraDevice = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)
        let cameraDeviceInput = try? AVCaptureDeviceInput(device: cameraDevice)
        guard let camera = cameraDeviceInput where captureSession.canAddInput(camera) 			else { return }
        captureSession.addInput(cameraDeviceInput)
        captureSession.startRunning()
    }    
```
---

**2** - Setup the Preview Layer (which is what you see when you movie your camera around):

```swift
previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
previewLayer.frame = view.bounds     
```

---

**3** - Setup the position of our "Pokemon" and add it to the preview layer

```swift
previewLayer.addSublayer(pokemon)
view.layer.addSublayer(previewLayer)
```

---

**4** - Setup the `CMMotionManager`

```swift
    private func setupMotionManager() {
        
        if motionManager.deviceMotionAvailable && motionManager.accelerometerAvailable {
            motionManager.deviceMotionUpdateInterval = 2.0 / 60.0
            motionManager.startDeviceMotionUpdatesToQueue(NSOperationQueue.currentQueue()!) { [unowned self] motion, error in
                
                if error != nil { print("wtf. \(error)"); return }
                guard let motion = motion else { print("Couln't unwrap motion"); return }
                
                self.quaternionX = motion.attitude.quaternion.x
                self.quaternionY = motion.attitude.quaternion.y
            }
        }
    }
```

---

**5** - Setup property observers on the `attitude.quaternion` properties on the `motion` object which allow us to update the "Pokemon" object on screen to make it appear as if it's moving with you.

```swift
var quaternionX: Double = 0.0 {
        didSet {
            if !foundTreasure { pokemon.center.y = (CGFloat(quaternionX) * view.bounds.size.width - 180) * 4.0 }
        }
    }
    var quaternionY: Double = 0.0 {
        didSet {
            if !foundTreasure { pokemon.center.x = (CGFloat(quaternionY) * view.bounds.size.height + 100) * 4.0 }
        }
    }

```
