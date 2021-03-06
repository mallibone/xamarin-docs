---
title: "Adding a Pan Gesture Recognizer"
description: "The pan gesture is used for detecting dragging and is implemented with the PanGestureRecognizer class. A common scenario for the pan gesture is to horizontally and vertically drag an image, so that all of the image content can be viewed when it's being displayed in a viewport smaller than the image dimensions. This is accomplished by moving the image within the viewport, and is demonstrated in this article."
ms.topic: article
ms.prod: xamarin
ms.assetid: 42CBD2CF-432D-4F19-A05E-D569BB7F8713
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 01/21/2016
---

# Adding a Pan Gesture Recognizer

_The pan gesture is used for detecting dragging and is implemented with the PanGestureRecognizer class. A common scenario for the pan gesture is to horizontally and vertically drag an image, so that all of the image content can be viewed when it's being displayed in a viewport smaller than the image dimensions. This is accomplished by moving the image within the viewport, and is demonstrated in this article._

## Overview

To make a user interface element draggable with the pan gesture, create a [`PanGestureRecognizer`](https://developer.xamarin.com/api/type/Xamarin.Forms.PanGestureRecognizer/) instance, handle the [`PanUpdated`](https://developer.xamarin.com/api/event/Xamarin.Forms.PanGestureRecognizer.PanUpdated/) event, and add the new gesture recognizer to the [`GestureRecognizers`](https://developer.xamarin.com/api/property/Xamarin.Forms.View.GestureRecognizers/) collection on the user interface element. The following code example shows a `PanGestureRecognizer` attached to an [`Image`](https://developer.xamarin.com/api/type/Xamarin.Forms.Image/) element:

```csharp
var panGesture = new PanGestureRecognizer();
panGesture.PanUpdated += (s, e) => {
  // Handle the pan
};
image.GestureRecognizers.Add(panGesture);
```

This can also be achieved in XAML, as shown in the following code example:

```xaml
<Image Source="MonoMonkey.jpg">
  <Image.GestureRecognizers>
    <PanGestureRecognizer PanUpdated="OnPanUpdated" />
  </Image.GestureRecognizers>
</Image>
```

The code for the `OnPanUpdated` event handler is then added to the code-behind file:

```csharp
void OnPanUpdated (object sender, PanUpdatedEventArgs e)
{
  // Handle the pan
}
```

> [!NOTE]
> Correct panning on Android requires the [Xamarin.Forms 2.1.0-pre1 NuGet package](https://www.nuget.org/packages/Xamarin.Forms/2.1.0.6501-pre1) at a minimum.

## Creating a Pan Container

This section contains a generalized helper class that performs freeform panning, which is typically suited to navigating within images or maps. Handling the pan gesture to perform a drag operation requires some math to transform the user interface. This math is used to drag only within the bounds of the wrapped user interface element. The following code example shows the `PanContainer` class:

```csharp
public class PanContainer : ContentView
{
  double x, y;

  public PanContainer ()
  {
    // Set PanGestureRecognizer.TouchPoints to control the
    // number of touch points needed to pan
    var panGesture = new PanGestureRecognizer ();
    panGesture.PanUpdated += OnPanUpdated;
    GestureRecognizers.Add (panGesture);
  }

  void OnPanUpdated (object sender, PanUpdatedEventArgs e)
  {
    ...
  }
}
```

This class can be wrapped around a user interface element so that the pan gesture will drag the wrapped user interface element. The following XAML code example shows the `PanContainer` wrapping an [`Image`](https://developer.xamarin.com/api/type/Xamarin.Forms.Image/) element:

```xaml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
			 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
			 xmlns:local="clr-namespace:PanGesture"
			 x:Class="PanGesture.HomePage">
	<ContentPage.Content>
		<AbsoluteLayout>
			<local:PanContainer>
				<Image Source="MonoMonkey.jpg" WidthRequest="1024" HeightRequest="768" />
			</local:PanContainer>
		</AbsoluteLayout>
	</ContentPage.Content>
</ContentPage>
```

The following code example shows how the `PanContainer` wraps an [`Image`](https://developer.xamarin.com/api/type/Xamarin.Forms.Image/) element in a C# page:

```csharp
public class HomePageCS : ContentPage
{
  public HomePageCS ()
  {
    Content = new AbsoluteLayout {
      Padding = new Thickness (20),
      Children = {
        new PanContainer {
          Content = new Image {
            Source = ImageSource.FromFile ("MonoMonkey.jpg"),
            WidthRequest = 1024,
            HeightRequest = 768
          }
        }
      }
    };
  }
}
```

In both examples, the [`WidthRequest`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.WidthRequest/) and [`HeightRequest`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.HeightRequest/) properties are set to the width and height values of the image being displayed.

When the [`Image`](https://developer.xamarin.com/api/type/Xamarin.Forms.Image/) element receives a pan gesture, the displayed image will be dragged. The drag is performed by the `PanContainer.OnPanUpdated` method, which is shown in the following code example:

```csharp
void OnPanUpdated (object sender, PanUpdatedEventArgs e)
{
  switch (e.StatusType) {
  case GestureStatus.Running:
    // Translate and ensure we don't pan beyond the wrapped user interface element bounds.
    Content.TranslationX =
      Math.Max (Math.Min (0, x + e.TotalX), -Math.Abs (Content.Width - App.ScreenWidth));
    Content.TranslationY =
      Math.Max (Math.Min (0, y + e.TotalY), -Math.Abs (Content.Height - App.ScreenHeight));
    break;

  case GestureStatus.Completed:
    // Store the translation applied during the pan
    x = Content.TranslationX;
    y = Content.TranslationY;
    break;
  }
}
```

This method updates the viewable content of the wrapped user interface element, based on the user's pan gesture. This is achieved by using the values of the [`TotalX`](https://developer.xamarin.com/api/property/Xamarin.Forms.PanUpdatedEventArgs.TotalX/) and [`TotalY`](https://developer.xamarin.com/api/property/Xamarin.Forms.PanUpdatedEventArgs.TotalY/) properties of the [`PanUpdatedEventArgs`](https://developer.xamarin.com/api/type/Xamarin.Forms.PanUpdatedEventArgs/) instance to calculate the direction and distance of the pan. The `App.ScreenWidth` and `App.ScreenHeight` properties provide the height and width of the viewport, and are set to the screen width and screen height values of the device by the respective platform-specific projects. The wrapped user element is then dragged by setting its [`TranslationX`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.TranslationX/) and [`TranslationY`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.TranslationY/) properties to the calculated values.

When panning content in an element that does not occupy the full screen, the height and width of the viewport can be obtained from the element's [`Height`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.Height/) and [`Width`](https://developer.xamarin.com/api/property/Xamarin.Forms.VisualElement.Width/) properties.

> [!NOTE]
> Displaying high-resolution images can greatly increase an app's memory footprint. Therefore, they should only be created when required and should be released as soon as the app no longer requires them. For more information, see [Optimize Image Resources](~/xamarin-forms/deploy-test/performance.md#optimizeimages).

## Summary

The pan gesture is used for detecting dragging and is implemented with the [`PanGestureRecognizer`](https://developer.xamarin.com/api/type/Xamarin.Forms.PanGestureRecognizer/) class.



## Related Links

- [PanGesture (sample)](https://developer.xamarin.com/samples/xamarin-forms/WorkingWithGestures/PanGesture/)
- [GestureRecognizer](https://developer.xamarin.com/api/type/Xamarin.Forms.GestureRecognizer/)
- [PanGestureRecognizer](https://developer.xamarin.com/api/type/Xamarin.Forms.PanGestureRecognizer/)
