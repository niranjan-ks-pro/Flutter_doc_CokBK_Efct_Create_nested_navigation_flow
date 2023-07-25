# Flutter_doc_CokBK_Efct_Create_nested_navigation_flow
 https://docs.flutter.dev/cookbook/effects/nested-nav#display-an-app-bar-for-the-setup-flow

Create a download button
========================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Create a download button](https://docs.flutter.dev/cookbook/effects/download-button)

Apps are filled with buttons that execute long-running behaviors. For example, a button might trigger a download, which starts a download process, receives data over time, and then provides access to the downloaded asset. It's helpful to show the user the progress of a long-running process, and the button itself is a good place to provide this feedback. In this recipe, you'll build a download button that transitions through multiple visual states, based on the status of an app download.

The following animation shows the app's behavior:

![The download button cycles through its stages](https://docs.flutter.dev/assets/images/docs/cookbook/effects/DownloadButton.gif)

[](https://docs.flutter.dev/cookbook/effects/download-button#define-a-new-stateless-widget)Define a new stateless widget
------------------------------------------------------------------------------------------------------------------------

Your button widget needs to change its appearance over time. Therefore, you need to implement your button with a custom stateless widget.

Define a new stateless widget called `DownloadButton`.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    // TODO:
    return const SizedBox();
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#define-the-buttons-possible-visual-states)Define the button's possible visual states
-------------------------------------------------------------------------------------------------------------------------------------------------

The download button's visual presentation is based on a given download status. Define the possible states of the download, and then update `DownloadButton` to accept a `DownloadStatus` and a `Duration` for how long the button should take to animate from one status to another.

content_copy

```
enum DownloadStatus {
  notDownloaded,
  fetchingDownload,
  downloading,
  downloaded,
}

@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.transitionDuration = const Duration(
      milliseconds: 500,
    ),
  });

  final DownloadStatus status;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    // TODO: We'll add more to this later.
    return const SizedBox();
  }
}
```

info Note: Each time you define a custom widget, you must decide whether all relevant information is provided to that widget from its parent or if that widget orchestrates the application behavior within itself. For example, `DownloadButton` could receive the current `DownloadStatus` from its parent, or the `DownloadButton` could orchestrate the download process itself within its `State` object. For most widgets, the best answer is to pass the relevant information into the widget from its parent, rather than manage behavior within the widget. By passing in all the relevant information, you ensure greater reusability for the widget, easier testing, and easier changes to application behavior in the future.

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-button-shape)Display the button shape
--------------------------------------------------------------------------------------------------------------

The download button changes its shape based on the download status. The button displays a grey, rounded rectangle during the `notDownloaded` and `downloaded` states. The button displays a transparent circle during the `fetchingDownload` and `downloading` states.

Based on the current `DownloadStatus`, build an `AnimatedContainer` with a `ShapeDecoration` that displays a rounded rectangle or a circle.

Consider defining the shape's widget tree in a separated `Stateless` widget so that the main `build()` method remains simple, allowing for the additions that follow. Instead of creating a function to return a widget, like `Widget _buildSomething() {}`, always prefer creating a `StatelessWidget` or a `StatefulWidget` which is more performant. More considerations on this can be found in the [documentation](https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html) or in a dedicated video in the Flutter [YouTube channel](https://www.youtube.com/watch?v=IOyq-eTRhvo).

For now, the `AnimatedContainer` child is just a `SizedBox` because we will come back at it in another step.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.transitionDuration = const Duration(
      milliseconds: 500,
    ),
  });

  final DownloadStatus status;
  final Duration transitionDuration;

  bool get _isDownloading => status == DownloadStatus.downloading;

  bool get _isFetching => status == DownloadStatus.fetchingDownload;

  bool get _isDownloaded => status == DownloadStatus.downloaded;

  @override
  Widget build(BuildContext context) {
    return ButtonShapeWidget(
      transitionDuration: transitionDuration,
      isDownloaded: _isDownloaded,
      isDownloading: _isDownloading,
      isFetching: _isFetching,
    );
  }
}

@immutable
class ButtonShapeWidget extends StatelessWidget {
  const ButtonShapeWidget({
    super.key,
    required this.isDownloading,
    required this.isDownloaded,
    required this.isFetching,
    required this.transitionDuration,
  });

  final bool isDownloading;
  final bool isDownloaded;
  final bool isFetching;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    var shape = const ShapeDecoration(
      shape: StadiumBorder(),
      color: CupertinoColors.lightBackgroundGray,
    );

    if (isDownloading || isFetching) {
      shape = ShapeDecoration(
        shape: const CircleBorder(),
        color: Colors.white.withOpacity(0),
      );
    }

    return AnimatedContainer(
      duration: transitionDuration,
      curve: Curves.ease,
      width: double.infinity,
      decoration: shape,
      child: const SizedBox(),
    );
  }
}
```

You might wonder why you need a `ShapeDecoration` widget for a transparent circle, given that it's invisible. The purpose of the invisible circle is to orchestrate the desired animation. The `AnimatedContainer` begins with a rounded rectangle. When the `DownloadStatus` changes to `fetchingDownload`, the `AnimatedContainer` needs to animate from a rounded rectangle to a circle, and then fade out as the animation takes place. The only way to implement this animation is to define both the beginning shape of a rounded rectangle and the ending shape of a circle. But, you don't want the final circle to be visible, so you make it transparent, which causes an animated fade-out.

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-button-text)Display the button text
------------------------------------------------------------------------------------------------------------

The `DownloadButton` displays `GET` during the `notDownloaded` phase, `OPEN` during the `downloaded` phase, and no text in between.

Add widgets to display text during each download phase, and animate the text's opacity in between. Add the text widget tree as a child of the `AnimatedContainer` in the button wrapper widget.

content_copy

```
@immutable
class ButtonShapeWidget extends StatelessWidget {
  const ButtonShapeWidget({
    super.key,
    required this.isDownloading,
    required this.isDownloaded,
    required this.isFetching,
    required this.transitionDuration,
  });

  final bool isDownloading;
  final bool isDownloaded;
  final bool isFetching;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    var shape = const ShapeDecoration(
      shape: StadiumBorder(),
      color: CupertinoColors.lightBackgroundGray,
    );

    if (isDownloading || isFetching) {
      shape = ShapeDecoration(
        shape: const CircleBorder(),
        color: Colors.white.withOpacity(0),
      );
    }

    return AnimatedContainer(
      duration: transitionDuration,
      curve: Curves.ease,
      width: double.infinity,
      decoration: shape,
      child: Padding(
        padding: const EdgeInsets.symmetric(vertical: 6),
        child: AnimatedOpacity(
          duration: transitionDuration,
          opacity: isDownloading || isFetching ? 0.0 : 1.0,
          curve: Curves.ease,
          child: Text(
            isDownloaded ? 'OPEN' : 'GET',
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.labelLarge?.copyWith(
                  fontWeight: FontWeight.bold,
                  color: CupertinoColors.activeBlue,
                ),
          ),
        ),
      ),
    );
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#display-a-spinner-while-fetching-download)Display a spinner while fetching download
------------------------------------------------------------------------------------------------------------------------------------------------

During the `fetchingDownload` phase, the `DownloadButton` displays a radial spinner. This spinner fades in from the `notDownloaded` phase and fades out to the `fetchingDownload` phase.

Implement a radial spinner that sits on top of the button shape and fades in and out at the appropriate times.

We have removed the `ButtonShapeWidget`'s constructor to keep the focus on its build method and the `Stack` widget we've added.

content_copy

```
@override
Widget build(BuildContext context) {
  return GestureDetector(
    onTap: _onPressed,
    child: Stack(
      children: [
        ButtonShapeWidget(
          transitionDuration: transitionDuration,
          isDownloaded: _isDownloaded,
          isDownloading: _isDownloading,
          isFetching: _isFetching,
        ),
        Positioned.fill(
          child: AnimatedOpacity(
            duration: transitionDuration,
            opacity: _isDownloading || _isFetching ? 1.0 : 0.0,
            curve: Curves.ease,
            child: ProgressIndicatorWidget(
              downloadProgress: downloadProgress,
              isDownloading: _isDownloading,
              isFetching: _isFetching,
            ),
          ),
        ),
      ],
    ),
  );
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-progress-and-a-stop-button-while-downloading)Display the progress and a stop button while downloading
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

After the `fetchingDownload` phase is the `downloading` phase. During the `downloading` phase, the `DownloadButton` replaces the radial progress spinner with a growing radial progress bar. The `DownloadButton` also displays a stop button icon so that the user can cancel an in-progress download.

Add a progress property to the `DownloadButton` widget, and then update the progress display to switch to a radial progress bar during the `downloading` phase.

Next, add a stop button icon at the center of the radial progress bar.

content_copy

```
@override
Widget build(BuildContext context) {
  return GestureDetector(
    onTap: _onPressed,
    child: Stack(
      children: [
        ButtonShapeWidget(
          transitionDuration: transitionDuration,
          isDownloaded: _isDownloaded,
          isDownloading: _isDownloading,
          isFetching: _isFetching,
        ),
        Positioned.fill(
          child: AnimatedOpacity(
            duration: transitionDuration,
            opacity: _isDownloading || _isFetching ? 1.0 : 0.0,
            curve: Curves.ease,
            child: Stack(
              alignment: Alignment.center,
              children: [
                ProgressIndicatorWidget(
                  downloadProgress: downloadProgress,
                  isDownloading: _isDownloading,
                  isFetching: _isFetching,
                ),
                if (_isDownloading)
                  const Icon(
                    Icons.stop,
                    size: 14.0,
                    color: CupertinoColors.activeBlue,
                  ),
              ],
            ),
          ),
        ),
      ],
    ),
  );
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#add-button-tap-callbacks)Add button tap callbacks
--------------------------------------------------------------------------------------------------------------

The last detail that your `DownloadButton` needs is the button behavior. The button must do things when the user taps it.

Add widget properties for callbacks to start a download, cancel a download, and open a download.

Finally, wrap `DownloadButton`'s existing widget tree with a `GestureDetector` widget, and forward the tap event to the corresponding callback property.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.downloadProgress = 0,
    required this.onDownload,
    required this.onCancel,
    required this.onOpen,
    this.transitionDuration = const Duration(milliseconds: 500),
  });

  final DownloadStatus status;
  final double downloadProgress;
  final VoidCallback onDownload;
  final VoidCallback onCancel;
  final VoidCallback onOpen;
  final Duration transitionDuration;

  bool get _isDownloading => status == DownloadStatus.downloading;

  bool get _isFetching => status == DownloadStatus.fetchingDownload;

  bool get _isDownloaded => status == DownloadStatus.downloaded;

  void _onPressed() {
    switch (status) {
      case DownloadStatus.notDownloaded:
        onDownload();
        break;
      case DownloadStatus.fetchingDownload:
        // do nothing.
        break;
      case DownloadStatus.downloading:
        onCancel();
        break;
      case DownloadStatus.downloaded:
        onOpen();
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _onPressed,
      child: const Stack(
        children: [
          /* ButtonShapeWidget and progress indicator */
        ],
      ),
    );
  }
}
```

Congratulations! You have a button that changes its display depending on which phase the button is in: not downloaded, fetching download, downloading, and downloaded. Now, the user can tap to start a download, tap to cancel an in-progress download, and tap to open a completed download.Create a nested navigation flow
===============================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Create a nested navigation flow](https://docs.flutter.dev/cookbook/effects/nested-nav)

Apps accumulate dozens and then hundreds of routes over time. Some of your routes make sense as top-level (global) routes. For example, "/", "profile", "contact", "social_feed" are all possible top-level routes within your app. But, imagine that you defined every possible route in your top-level `Navigator` widget. The list would be very long, and many of these routes would be better handled nested within another widget.

Consider an Internet of Things (IoT) setup flow for a wireless light bulb that you control with your app. This setup flow consists of 4 pages: find nearby bulbs, select the bulb that you want to add, add the bulb, and then complete the setup. You could orchestrate this behavior from your top-level `Navigator` widget. However, it makes more sense to define a second, nested `Navigator` widget within your `SetupFlow` widget, and let the nested `Navigator` take ownership over the 4 pages in the setup flow. This delegation of navigation facilitates greater local control, which is generally preferable when developing software.

The following animation shows the app's behavior:

![Gif showing the nested "setup" flow](https://docs.flutter.dev/assets/images/docs/cookbook/effects/NestedNavigator.gif)

In this recipe, you implement a four-page IoT setup flow that maintains its own navigation nested beneath the top-level `Navigator` widget.

[](https://docs.flutter.dev/cookbook/effects/nested-nav#prepare-for-navigation)Prepare for navigation
-----------------------------------------------------------------------------------------------------

This IoT app has two top-level screens, along with the setup flow. Define these route names as constants so that they can be referenced within code.

content_copy

```
const routeHome = '/';
const routeSettings = '/settings';
const routePrefixDeviceSetup = '/setup/';
const routeDeviceSetupStart = '/setup/$routeDeviceSetupStartPage';
const routeDeviceSetupStartPage = 'find_devices';
const routeDeviceSetupSelectDevicePage = 'select_device';
const routeDeviceSetupConnectingPage = 'connecting';
const routeDeviceSetupFinishedPage = 'finished';
```

The home and settings screens are referenced with static names. The setup flow pages, however, use two paths to create their route names: a `/setup/` prefix followed by the name of the specific page. By combining the two paths, your `Navigator` can determine that a route name is intended for the setup flow without recognizing all the individual pages associated with the setup flow.

The top-level `Navigator` isn't responsible for identifying individual setup flow pages. Therefore, your top-level `Navigator` needs to parse the incoming route name to identify the setup flow prefix. Needing to parse the route name means that you can't use the `routes` property of your top-level `Navigator`. Instead, you must provide a function for the `onGenerateRoute` property.

Implement `onGenerateRoute` to return the appropriate widget for each of the three top-level paths.

content_copy

```
onGenerateRoute: (settings) {
  late Widget page;
  if (settings.name == routeHome) {
    page = const HomeScreen();
  } else if (settings.name == routeSettings) {
    page = const SettingsScreen();
  } else if (settings.name!.startsWith(routePrefixDeviceSetup)) {
    final subRoute =
        settings.name!.substring(routePrefixDeviceSetup.length);
    page = SetupFlow(
      setupPageRoute: subRoute,
    );
  } else {
    throw Exception('Unknown route: ${settings.name}');
  }

  return MaterialPageRoute<dynamic>(
    builder: (context) {
      return page;
    },
    settings: settings,
  );
},
```

Notice that the home and settings routes are matched with exact route names. However, the setup flow route condition only checks for a prefix. If the route name contains the setup flow prefix, then the rest of the route name is ignored and passed on to the `SetupFlow` widget to process. This splitting of the route name is what allows the top-level `Navigator` to be agnostic toward the various subroutes within the setup flow.

Create a stateful widget called `SetupFlow` that accepts a route name.

content_copy

```
class SetupFlow extends StatefulWidget {
  const SetupFlow({
    super.key,
    required this.setupPageRoute,
  });

  final String setupPageRoute;

  @override
  SetupFlowState createState() => SetupFlowState();
}

class SetupFlowState extends State<SetupFlow> {
  //...
}
```

[](https://docs.flutter.dev/cookbook/effects/nested-nav#display-an-app-bar-for-the-setup-flow)Display an app bar for the setup flow
-----------------------------------------------------------------------------------------------------------------------------------

The setup flow displays a persistent app bar that appears across all pages.

Return a `Scaffold` widget from your `SetupFlow` widget's `build()` method, and include the desired `AppBar` widget.

content_copy

```
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: _buildFlowAppBar(),
    body: const SizedBox(),
  );
}

PreferredSizeWidget _buildFlowAppBar() {
  return AppBar(
    title: const Text('Bulb Setup'),
  );
}
```

The app bar displays a back arrow and exits the setup flow when the back arrow is pressed. However, exiting the flow causes the user to lose all progress. Therefore, the user is prompted to confirm whether they want to exit the setup flow.

Prompt the user to confirm exiting the setup flow, and ensure that the prompt appears when the user presses the hardware back button on Android.

content_copy

```
Future<void> _onExitPressed() async {
  final isConfirmed = await _isExitDesired();

  if (isConfirmed && mounted) {
    _exitSetup();
  }
}

Future<bool> _isExitDesired() async {
  return await showDialog<bool>(
          context: context,
          builder: (context) {
            return AlertDialog(
              title: const Text('Are you sure?'),
              content: const Text(
                  'If you exit device setup, your progress will be lost.'),
              actions: [
                TextButton(
                  onPressed: () {
                    Navigator.of(context).pop(true);
                  },
                  child: const Text('Leave'),
                ),
                TextButton(
                  onPressed: () {
                    Navigator.of(context).pop(false);
                  },
                  child: const Text('Stay'),
                ),
              ],
            );
          }) ??
      false;
}

void _exitSetup() {
  Navigator.of(context).pop();
}

@override
Widget build(BuildContext context) {
  return WillPopScope(
    onWillPop: _isExitDesired,
    child: Scaffold(
      appBar: _buildFlowAppBar(),
      body: const SizedBox(),
    ),
  );
}

PreferredSizeWidget _buildFlowAppBar() {
  return AppBar(
    leading: IconButton(
      onPressed: _onExitPressed,
      icon: const Icon(Icons.chevron_left),
    ),
    title: const Text('Bulb Setup'),
  );
}
```

When the user taps the back arrow in the app bar, or presses the back button on Android, an alert dialog pops up to confirm that the user wants to leave the setup flow. If the user presses Leave, then the setup flow pops itself from the top-level navigation stack. If the user presses Stay, then the action is ignored.

You might notice that the `Navigator.pop()` is invoked by both the Leave and Stay buttons. To be clear, this `pop()` action pops the alert dialog off the navigation stack, not the setup flow.

[](https://docs.flutter.dev/cookbook/effects/nested-nav#generate-nested-routes)Generate nested routes
-----------------------------------------------------------------------------------------------------

The setup flow's job is to display the appropriate page within the flow.

Add a `Navigator` widget to `SetupFlow`, and implement the `onGenerateRoute` property.

content_copy

```
final _navigatorKey = GlobalKey<NavigatorState>();

void _onDiscoveryComplete() {
  _navigatorKey.currentState!.pushNamed(routeDeviceSetupSelectDevicePage);
}

void _onDeviceSelected(String deviceId) {
  _navigatorKey.currentState!.pushNamed(routeDeviceSetupConnectingPage);
}

void _onConnectionEstablished() {
  _navigatorKey.currentState!.pushNamed(routeDeviceSetupFinishedPage);
}

@override
Widget build(BuildContext context) {
  return WillPopScope(
    onWillPop: _isExitDesired,
    child: Scaffold(
      appBar: _buildFlowAppBar(),
      body: Navigator(
        key: _navigatorKey,
        initialRoute: widget.setupPageRoute,
        onGenerateRoute: _onGenerateRoute,
      ),
    ),
  );
}

Route _onGenerateRoute(RouteSettings settings) {
  late Widget page;
  switch (settings.name) {
    case routeDeviceSetupStartPage:
      page = WaitingPage(
        message: 'Searching for nearby bulb...',
        onWaitComplete: _onDiscoveryComplete,
      );
      break;
    case routeDeviceSetupSelectDevicePage:
      page = SelectDevicePage(
        onDeviceSelected: _onDeviceSelected,
      );
      break;
    case routeDeviceSetupConnectingPage:
      page = WaitingPage(
        message: 'Connecting...',
        onWaitComplete: _onConnectionEstablished,
      );
      break;
    case routeDeviceSetupFinishedPage:
      page = FinishedPage(
        onFinishPressed: _exitSetup,
      );
      break;
  }

  return MaterialPageRoute<dynamic>(
    builder: (context) {
      return page;
    },
    settings: settings,
  );
}
```

The `_onGenerateRoute` function works the same as for a top-level `Navigator`. A `RouteSettings` object is passed into the function, which includes the route's `name`. Based on that route name, one of four flow pages is returned.

The first page, called `find_devices`, waits a few seconds to simulate network scanning. After the wait period, the page invokes its callback. In this case, that callback is `_onDiscoveryComplete`. The setup flow recognizes that, when device discovery is complete, the device selection page should be shown. Therefore, in `_onDiscoveryComplete`, the `_navigatorKey` instructs the nested `Navigator` to navigate to the `select_device` page.

The `select_device` page asks the user to select a device from a list of available devices. In this recipe, only one device is presented to the user. When the user taps a device, the `onDeviceSelected` callback is invoked. The setup flow recognizes that, when a device is selected, the connecting page should be shown. Therefore, in `_onDeviceSelected`, the `_navigatorKey` instructs the nested `Navigator` to navigate to the `"connecting"` page.

The `connecting` page works the same way as the `find_devices` page. The `connecting` page waits for a few seconds and then invokes its callback. In this case, the callback is `_onConnectionEstablished`. The setup flow recognizes that, when a connection is established, the final page should be shown. Therefore, in `_onConnectionEstablished`, the `_navigatorKey` instructs the nested `Navigator` to navigate to the `finished` page.

The `finished` page provides the user with a Finish button. When the user taps Finish, the `_exitSetup` callback is invoked, which pops the entire setup flow off the top-level `Navigator` stack, taking the user back to the home screen.

Congratulations! You implemented nested navigation with four subroutes.
