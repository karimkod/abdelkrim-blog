---
title: Toolbox For Porting Your Flutter Mobile Application to The Web
description: Unlock the potential of your Flutter mobile app on the web. Dive into the techniques and best practices of seamlessly porting your Flutter app to the web using Dart. Explore the intricacies of adapting UI components, optimizing performance, and ensuring a consistent user experience. Join me as we bridge the gap between mobile and web, harnessing the power of Flutter and Dart to extend your app's reach and impact.
categories: Flutter
featured_image: /assets/img/flutter-web-portability/1_gnHqSRM2dV50E7xAoe1Lew.webp
image: /assets/img/flutter-web-portability/1_gnHqSRM2dV50E7xAoe1Lew.webp
featured_image_alt: An illustration of porting a flutter mobile app to the web
comments: true
---

Since Google announced that Flutter web is stable starting from v2.0.0, a lot of business and application owners wanted to have a version of their already developed Flutter application ported to the web, since the web is easily accessible and Progressive Web Applications (PWA) offer an experience close to native apps.

This article doesn’t discuss UI matters (responsiveness, UI/UX…), it only tackles the problems we may encounter for running the app and its functionalities on the web platform.

Since that announcement, I’ve been on several projects of porting Flutter mobile applications to the web. I have organized a set of steps and techniques that I follow every time I have a Flutter application to port to the web. These steps make it easier for me to detect what is not supported on the web and, depending on the situation, fix it or change it.

## Application Analysis :

The first step is analyzing the application, the goal of this step is to check if any dependencies are not supported on the web. The first thing to do is try to run for the web by executing

`flutter run -d chrome`
Or change the target to Chrome and run it from your IDE.

This step is just to discover if there is anything that’s preventing the compilation to the web (probably the use of a dart library that’s not supported on the web. If any, refer to 2 —The toolbox part of this article) If the application ran properly it doesn’t mean that it’s working on the web and our job is done! Some issues may appear at runtime.

Now, if the application ran without any compilation error we proceed with the second phase of the application analysis, which is analyzing the dependencies. There are two kinds of dependencies that may interfere with our goal :

### Dart libraries :

Sometimes developers use dart core libraries for some specific functionalities like using the File type (dart:io) and JSON encoding/decoding (dart:convert). Well, some of these core libraries are only supported in dart native platforms and not on the web and vice-versa. So, part of our job is to check if we are referring to any of these in our code. You can check the compatibility of the library with the target platform on the official page.

### External packages :

The other resort for developers to find a tool that fits their needs is the famous pub.dev, we refer to the package and the version we want to use in the pubspec.yaml file. So, our job is to go through that list (manually for now) and check if the package is web-compatible. We can know by checking the pub.dev page of that package and checking the platform tags of the version we depend on. For instance, the 7.3.1 version of flutter_bloc package is supported in Android, IOS, Linux, macOS, Web, and Windows and we can see that in the tags (we can change the version of the package in the versions tab):

After we have identified the blockers we start tackling them one by one using the appropriate tool each time.

## The Toolbox :

In this part, we will go through the tools that will help us tackle the blockers we identified. The usage of each tool depends on your situation. The goal of this enumeration is to put the tools we have in front of us.

### Alternatives :

The first smart step we can take is finding alternatives. This can either be a recent version of the same package or a completely different package. Most packages’ authors have upgraded their packages to support the web, it’s a good reflex to go check if the packages have a recent version that supports the web or, sometimes, it refers to another package that’s considered a fork/upgrade of it that does.

Another factor that comes into play when replacing a package (especially if we didn’t make a boundary between it and our code) is the time and effort of the substitution.

Sometimes, we can’t find that alternative or it’s costly to adopt it. Then, It’s practical to keep the mobile code as it is and implement something web-specific.

### Platform-specific Code :

Flutter and dart offer many ways to write and selectively execute platform-specific code.

To select which platform code to execute, dart has two mechanisms that help us decide in case of the web:

**kIsWeb flag:**
The kIsWeb is a boolean that will be set to true if your application is running on a web platform. This boolean is pretty useful to execute code that’s specific to the web.

**Conditional Imports:**
Sometimes, we use libraries that prevent our code from even compiling to a specific platform (case of dart platform libraries — see above). The solution to that is to conditionally import a library. Conditional imports (it’s polymorphism for packages) let you import a package instead of another one if a condition is met, though, they need to provide the same interfaces (just like polymorphism) so that the substitution keeps the code valid.

If, for example, we have package abc that is supported on mobile and xyz that’s supported on the web and they provide the same interfaces. We can write :

```dart
import 'package:abc/abc.dart'
if(dart.library.js) 'package:xyz/xyz.dart' // in case of web
```

I can’t go through all the details about it (for the article-length matter) but it’s useful to have that tool in our toolbox, here is an article that explains it in detail alongside a use case.

Now, having mechanisms to help us select what platform code to execute, we can use the tools that help us write web-specific code.

**Using a Javascript Code/Library:**
As you may know, dart supports Javascript interoperability which allows it to deal with JS code. We can use this to our advantage. Sometimes we find out that the official flutter SDK we are using (in my case it was braintree payment SDK by the time of the writing) doesn’t support flutter web ‘officially’. And that solution has an official JS library. We can, using js interoperability; use that js library on the flutter web.

The js package can help us do that. It is used by defining dart classes/functions that are equivalent to the JS code and letting their implementation be provided from javascript (by using the external keyword) then we mark the dart code with annotations. For example, we can use JSON.stringify from dart by, first, defining the equivalent dart code:

```dart
@JS()
library stringify;
import 'package:js/js.dart';
@JS('JSON.stringify')
external String stringify(Object obj);
```

and then just call the stringify function in a dart code. For an advanced example, refer to the charjs.dart repo (I often use it as my reference).

### 3 - Extending packages :

Now, if you still want to use one flutter package for all platforms (for any reason you have) or you think that changing to another package can be costly. You can still use the open-heart surgery option (thanks to the open-source nature of flutter and flutter packages).

This solution may seem complicated but once we know the basics of creating flutter packages and read some useful articles about it (with good repository references and examples) it becomes more accessible to do it (depending on our knowledge of the target platform) All we have to do is fork the package and start working on it.

Unfortunately, I can’t go through all the details about extending packages but the following list contains the resources (articles and git repositories) I used to get the job done :

- [Official documentation of writing packages.](https://docs.flutter.dev/development/packages-and-plugins/developing-packages#plugin)
- [Harry Terkelsen](https://medium.com/@harry.terkelsen?source=post_page-----da1ce4f89301--------------------------------)’s articles about adding web support for url_launcher package: using Method Channels [Part 1](https://medium.com/flutter/how-to-write-a-flutter-web-plugin-5e26c689ea1) OR using Federated plugin [Part 2](https://medium.com/flutter/how-to-write-a-flutter-web-plugin-part-2-afdddb69ece6).
- [url_launcher](https://github.com/flutter/plugins/tree/main/packages/url_launcher/url_launcher) repository is a good reference.

I have personally used this to extend the geocoding package, which was a federated plugin; to support the web.

## Conclusion

Now, we did what’s necessary but the only way to know if our job is done is by going through an efficient test process (favorably, automatic, or else manual).
Having this toolbox in hand can, hopefully; be useful to you. If you have any feedback/comments/Critics or you just wanna connect. please reach out to me through my LinkedIn or leave them in the comment section.
