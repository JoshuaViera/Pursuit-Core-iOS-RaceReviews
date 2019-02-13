# Pursuit-Core-iOS-RaceReviews
RaceReviews uses the Firebase Authentication and Cloud Firestore database to create users. Authenticated users are able to add an annotation on a MapView along with a race review.

## Lessons Links

- Creating a .xib file [lesson link](https://github.com/joinpursuit/Pursuit-Core-iOS/tree/master/units/unit05/lesson-06-xibs) 
- Cocoapods [lesson link](https://github.com/joinpursuit/Pursuit-Core-iOS/tree/master/units/unit05/lesson-07-cocoapods) 
- Firebase Intro [lesson link](https://github.com/joinpursuit/Pursuit-Core-iOS/tree/master/units/unit05/lesson-08-intro-to-firebase-baas)  

## Installation Procedures for this app

- clone this repo 
- all Pods are included 
- open up the RaceReviews.xcworkspace project and run the app on your simulator or device

**Instructions on replacing the GoogleService-Info.plist file in this repo with your own**   
1. clone this repo
1. make sure the app runs on a simulator or device
1. sign out of the RaceReviews app 
1. delete the GoogleService-Info.plist from Xcode => “Move to Trash”
1. navigate to https://console.firebase.google.com/u/1/ , click on the RaceReviews app, then go to the **Project Settings** and download and drag your **GoogleService-Info.plist** to Xcode beneath the **Info.plist**
1. make sure you **save as** **GoogleService-Info.plist** and not **GoogleService-Info (1).plist** in the case a file already exist at the download path 
1. make sure **Copy items if needed**, **Create groups** and **Add to targets** are checked
1. run the app in the simulator or on your device
1. create an new authentication account and verify the user was created under “Users” in the Authentication section of the Firebase dashboard


## Firebase Cocoapods used 

```
pod 'Firebase/Core'
pod 'Firebase/Auth'
pod 'Firebase/Firestore'
pod 'Firebase/Storage'
```

**Checklist**  
- [x] create the Xcode project
- [x] create firebase console project 
- [x] add google service plist file to xcode
- [x] add firebase sdk to the xcode project using cocoapods (firebase/core, /auth, /firestore, /storage)
- [x] confirm firebase installation in the firebase console after importing firebase into the Xcode project
- [x] add email/password authentication to the firebase project
- [x] create .xib login view
- [x] user can create an authenticated account using their email and password
- [x] user is created in cloud firestore database after successfullly creating an account
- [x] present the race reviews tab controller if login is successful
- [x] architect login flow in the app delegate base on current user sign in state
- [x] user can sign out of the app 
- [x] present the login view controller when the user signs out
- [x] existing user can sign in to the app 
- [x] race review creation UI includes textfield to enter name and text view for review (firebase database)
- [x] user can create a review (create)
- [ ] user can see an annotation on a MapView of their created review (read)
- [ ] user can udpate a review (only if userId == reviewerId) (update) 
- [ ] user can delete a review (only if userId == reviewerId) (delete) 

## Race Review Model 

```swift 
import Foundation
import CoreLocation

enum RaceType: CaseIterable {
  case swimming
  case biking
  case running
  case obstacle
  case other
}

struct RaceReview {
  let name: String
  let review: String
  let type: RaceType
  let lat: Double
  let lon: Double
  let reviewerId: String
  
  init(name: String, review: String, type: RaceType, lat: Double, lon: Double, reviewerId: String) {
    self.name = name
    self.review = review
    self.type = type
    self.lat = lat
    self.lon = lon
    self.reviewerId = reviewerId
  }
  
  init(dict: [String: Any]) {
    self.name = dict["name"] as? String ?? "no race name"
    self.review = dict["review"] as? String ?? "no race review"
    self.type = dict["type"] as? RaceType ?? .other
    self.lat = dict["lat"] as? Double ?? 0
    self.lon = dict["lon"] as? Double ?? 0
    self.reviewerId = dict["reviewerId"] as? String ?? "no reviewerId"
  }
  
  public var coordinate: CLLocationCoordinate2D {
    return CLLocationCoordinate2DMake(lat, lon)
  }
}
```

## Cloud Firestore Security Rules 

**Test Mode: anyone with the Database Reference can read and write**   
```javascript
// Allow read/write access to all users under any conditions
// Warning: **NEVER** use this rule set in production; it allows
// anyone to overwrite your entire database.
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

**Locked Mode**  
```javascript 
// Deny read/write access to all users under any conditions
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

**Only signed in users can read, write**   
```javascript 
// Allow read/write access on all documents to any user signed in to the application
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth.uid != null;
    }
  }
}
```

**Another common pattern is to make sure users can only read and write their own data:**   
```javascript 
service cloud.firestore {
  match /databases/{database}/documents {
    // Make sure the uid of the requesting user matches name of the user
    // document. The wildcard expression {userId} makes the userId variable
    // available in rules.
    match /users/{userId} {
      allow read, update, delete: if request.auth.uid == userId;
      allow create: if request.auth.uid != null;
    }
  }
}
