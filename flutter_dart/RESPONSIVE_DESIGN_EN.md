# Sprache/Language : [DE](RESPONSIVE_DESIGN.md) | [EN](RESPONSIVE_DESIGN_EN.md)
# 📱 Responsive Design Patterns in Flutter with Custom Breakpoints

Flutter is a UI toolkit platform that runs on many devices – from small phones to large desktops. To create user-friendly and scalable layouts, **Responsive Design** is essential.

This documentation shows a **custom breakpoint system** using **Dart Extensions** for `BuildContext` and
`Widget`.

---

## 🧠 1. Why Responsive Design?

Responsive Design enables:
- Consistent user experience across different screen sizes
- Dynamic layouts adapted to device types
- Maintainable and adaptable UI structures

---

## 🎯 2. What is a Breakpoint System?

A **Breakpoint** is a defined screen width at which the layout is adjusted. Typical categories:

### 📐 Typical Breakpoint Sizes (in Pixels)

| Device Type                    | Description                    | Typical Width (px)     |
| ------------------------------ | ------------------------------ | ---------------------- |
| **Small Smartphone**           | E.g., iPhone SE, older devices | `≤ 360`–`412` px       |
| **Large Smartphone**           | Current iPhones, Pixel, etc.   | `≥ 413` – `≤ 576` px   |
| **Tablet (Portrait)**          | iPad, Galaxy Tab (portrait)    | `≥ 577` – `≤ 768` px   |
| **Tablet (Landscape)**         | iPad landscape                 | `≈ 800` – `1024` px    |
| **Small Desktop / Notebook**   | 13–15″ devices                 | `≥ 992` – `≤ 1366` px  |
| **Full HD Desktop**            | Standard 1080p monitors        | `≥ 1366` – `≤ 1920` px |
| **4K Desktop**                 | UHD displays, large screens    | `> 1920` – `3840` px   |


>### 🧠 Notes:
>- Devices like the iPhone 14 Pro Max have a physical width of e.g. 430 px (logical pixels in Flutter).
>- Flutter uses logical pixels (not actual screen pixels), which makes breakpoints stable across devices.
>- When you use e.g. MediaQuery.of(context).size.width, you get logical width that is already scaled.

> This ensures that content doesn't appear overwhelming on small devices and doesn't look empty on large devices.

![Breakpoint Scale](assets/breakpoint.svg)

### ✅ Recommended Custom Breakpoints for Flutter (customizable):
```dart
class Breakpoints {
  static const double smallPhone = 360;
  static const double largePhone = 430;
  static const double tablet = 768;
  static const double desktop = 1024;
  static const double fullHD = 1440;
  static const double ultraHD = 1920;
}
```


## 🧩 3. Dart Extensions: Briefly Explained

Dart **Extensions** allow you to **functionally extend** existing classes like `BuildContext` or `Widget` without inheriting from them.

Example:

```dart
extension ResponsiveContext on BuildContext {
  double get width => MediaQuery.of(this).size.width;
}
```
Result:
```dart
final screenWidth = context.width;
```

## 🏗️ 4. Classes and Extensions
### 🔹 Breakpoints
Defines fixed threshold values (e.g., Breakpoints.tablet) for responsive decisions.

```dart
class Breakpoints {
  static const double tablet = 768;
  // ...
}
```

### 🔹 DeviceType & ResponsiveContext
Enables simple device detection:

```dart
if (context.isTabletOrLarger) {
  // Desktop layout
}
```

#### Additional Getters:

- isMobile, isTablet, isDesktop, deviceType
- responsive<T>(...) for value selection by DeviceType
- isDarkMode, isLightMode

### 🔹 ResponsiveWidget Extension
Makes widgets visible/invisible depending on DeviceType or dynamically adds padding, margin, constraints:

```dart
Text("Hello").showOnMobile(context)
Container().responsivePadding(context, mobile: EdgeInsets.all(8))
```

### 🔹 ResponsiveSize Utility
Dynamically calculates sizes, e.g., for text or percentages:
```dart
Text(
  "Headline",
  style: TextStyle(fontSize: ResponsiveSize.fontSize(context, desktop: 24, mobile: 16, defaultSize: 18)),
)
```

### 🔹 ResponsiveBuilder
Flexible variant of a builder for different devices:
```dart
ResponsiveBuilder(
  tablet: TabletLayout(),
  desktop: DesktopLayout(),
  defaultWidget: MobileLayout(),
)
```
or dynamically:
```dart
builder: (context, deviceType) {
if (deviceType == DeviceType.desktop) return DesktopView();
return MobileView();
}
```
## ✅ 5. Best Practices

| Recommendation                               | Description                                     |
| -------------------------------------------- | ----------------------------------------------- |
| 📐 Clear Breakpoint Definition               | Use constant values for reusability            |
| ♻️ Use Extensions                            | Improves readability and structure              |
| ⚖️ Not Too Many Layout Versions              | Keep tablet/desktop together when possible      |
| 💡 responsive(...) instead of if-else        | Avoids redundant code                          |
| 📦 `ResponsiveBuilder` for large components  | Replaces manual switching in the view          |


## 🧾 6. Summary
This system provides:

- Clearly structured device logic through DeviceType
- Flexible UI components controllable through Extensions
- Reusable utilities for layout, text sizes, padding
- Ideal for medium to large projects where responsiveness is not optional.

## Back to Content:
[Back to Starting Point](../README_EN.md)