# Adhan Dart / Flutter
![Dart CI](https://github.com/iamriajul/adhan-dart/workflows/Dart%20CI/badge.svg?branch=master)

Adhan Dart is a well tested and well documented library for calculating Islamic prayer times. Adhan Dart is written to be compatible with Dart and Dart Native / Flutter / Android / iOS / Web etc... devices of all api versions. It has a small method overhead.

All astronomical calculations are high precision equations directly from the book “Astronomical Algorithms” by Jean Meeus. This book is recommended by the Astronomical Applications Department of the U.S. Naval Observatory and the Earth System Research Laboratory of the National Oceanic and Atmospheric Administration.

Implementations of Adhan in other languages can be found in the parent repo Adhan.

## Usage

A simple usage example:

```dart
import 'package:adhan/adhan.dart';

main() {
  print('Kushtia Prayer Times');
  final kushtia = Coordinates(23.9088, 89.1220);
  final date = DateComponents(2020, 6, 12);
  final params = CalculationMethod.karachi.getParameters();
  params.madhab = Madhab.hanafi;
  final prayerTimes = PrayerTimes(kushtia, date, params);

  print(prayerTimes.fajr.timeZoneName);
  print(DateFormat.jm().format(prayerTimes.fajr));
  print(DateFormat.jm().format(prayerTimes.sunrise));
  print(DateFormat.jm().format(prayerTimes.dhuhr));
  print(DateFormat.jm().format(prayerTimes.asr));
  print(DateFormat.jm().format(prayerTimes.maghrib));
  print(DateFormat.jm().format(prayerTimes.isha));

  print('---');

  print('NewYork Prayer Times');
  final newYork = Coordinates(35.7750, -78.6336);
  final nyUtcOffset = Duration(hours: -4);
  final nyDate = DateComponents(2015, 7, 12);
  final nyParams = CalculationMethod.north_america.getParameters();
  nyParams.madhab = Madhab.hanafi;
  final nyPrayerTimes = PrayerTimes(newYork, nyDate, nyParams, utcOffset: nyUtcOffset);

  print(nyPrayerTimes.fajr.timeZoneName);
  print(DateFormat.jm().format(nyPrayerTimes.fajr));
  print(DateFormat.jm().format(nyPrayerTimes.sunrise));
  print(DateFormat.jm().format(nyPrayerTimes.dhuhr));
  print(DateFormat.jm().format(nyPrayerTimes.asr));
  print(DateFormat.jm().format(nyPrayerTimes.maghrib));
  print(DateFormat.jm().format(nyPrayerTimes.isha));
}
```

### Initialization parameters

#### Coordinates

Create a `Coordinates` object with the latitude and longitude for the location you want prayer times for.

```dart
final coordinates = new Coordinates(35.78056, -78.6389);
```

#### Date

The date parameter passed in should be an instance of the `DateComponents` object. The year, month, and day values need to be populated. All other values will be ignored. The year, month and day values should be for the  local date that you want prayer times for. These date values are expected to be for the Gregorian calendar. There's also a convenience method for converting a `java.util.Date` to `DateComponents`.

```dart
final date = DateComponents(2015, 11, 1);
// or
final date = DateComponents.from(DateTime.now());
```

#### Calculation parameters

The rest of the needed information is contained within the `CalculationParameters` class. Instead of manually initializing this class, it is recommended to use one of the pre-populated instances in the `CalculationMethod` class. You can then further customize the calculation parameters if needed.

```dart
final params = CalculationMethod.muslim_world_league.getParameters();
params.madhab = Madhab.hanafi;
params.adjustments.fajr = 2;
```

| Parameter | Description |
| --------- | ----------- |
| `method`    | CalculationMethod name |
| `fajrAngle` | Angle of the sun used to calculate Fajr |
| `ishaAngle` | Angle of the sun used to calculate Isha |
| `ishaInterval` | Minutes after Maghrib (if set, the time for Isha will be Maghrib plus ishaInterval) |
| `madhab` | Value from the Madhab object, used to calculate Asr |
| `highLatitudeRule` | Value from the HighLatitudeRule object, used to set a minimum time for Fajr and a max time for Isha |
| `adjustments` | JavaScript object with custom prayer time adjustments in minutes for each prayer time |

**CalculationMethod**

| Value | Description |
| ----- | ----------- |
| `muslim_world_league` | Muslim World League. Fajr angle: 18, Isha angle: 17 |
| `egyptian` | Egyptian General Authority of Survey. Fajr angle: 19.5, Isha angle: 17.5 |
| `karachi` | University of Islamic Sciences, Karachi. Fajr angle: 18, Isha angle: 18 |
| `umm_al_qura` | Umm al-Qura University, Makkah. Fajr angle: 18, Isha interval: 90. *Note: you should add a +30 minute custom adjustment for Isha during Ramadan.* |
| `dubai` | Method used in UAE. Fajr and Isha angles of 18.2 degrees. |
| `qatar` | Modified version of Umm al-Qura used in Qatar. Fajr angle: 18, Isha interval: 90. |
| `kuwait` | Method used by the country of Kuwait. Fajr angle: 18, Isha angle: 17.5 |
| `moonsighting_committee` | Moonsighting Committee. Fajr angle: 18, Isha angle: 18. Also uses seasonal adjustment values. |
| `singapore` | Method used by Singapore. Fajr angle: 20, Isha angle: 18. |
| `north_america` | Referred to as the ISNA method. This method is included for completeness but is not recommended. Fajr angle: 15, Isha angle: 15 |
| `kuwait` | Kuwait. Fajr angle: 18, Isha angle: 17.5 |
| `other` | Fajr angle: 0, Isha angle: 0. This is the default value for `method` when initializing a `CalculationParameters` object. |

**Madhab**

| Value | Description |
| ----- | ----------- |
| `shafi` | Earlier Asr time |
| `hanafi` | Later Asr time |

**HighLatitudeRule**

| Value | Description |
| ----- | ----------- |
| `middle_of_the_night` | Fajr will never be earlier than the middle of the night and Isha will never be later than the middle of the night |
| `seventh_of_the_night` | Fajr will never be earlier than the beginning of the last seventh of the night and Isha will never be later than the end of the first seventh of the night |
| `twilight_angle` | Similar to `SEVENTH_OF_THE_NIGHT`, but instead of 1/7, the fraction of the night used is fajrAngle/60 and ishaAngle/60 |


### Prayer Times

Once the `PrayerTimes` object has been initialized it will contain values for all five prayer times and the time for sunrise. The prayer times will be DateTime object instances initialized with Local values (converted from UTC via .toLocal method). To display these times for a different timezone than local a UTC Offset should be used, for example `Duration`.

```dart
final kushtiaUtcOffset = Duration(hours: 6);
final newYorkUtcOffset = Duration(hours: -4);
// then pass your offset to PrayerTimes like this:
final prayerTimes = PrayerTimes(coordinates, date, params, utcOffset: newYorkUtcOffset);
```


**Methods**

| name | return |
| ----- | ----------- |
| `timeForPrayer` | DateTime instance |
| `currentPrayer` | Prayer enum instance |
| `currentPrayerByDateTime` | Prayer enum instance |
| `nextPrayer` | Prayer enum instance |
| `nextPrayerByDateTime` | Prayer enum instance |

## TODO
1. add Qibla
2. add SunnahTimes

## Inspired / Adopted From
- [adhan-java](https://github.com/batoulapps/adhan-java/blob/master/README.md)

# Adhan Java

Adhan Java is a well tested and well documented library for calculating Islamic prayer times. Adhan Java is written to be compatible with Java and Android devices of all api versions. It compiles against Java 7 to ensure compatibility with Android. It has a small method overhead, and has no external dependencies.

All astronomical calculations are high precision equations directly from the book [“Astronomical Algorithms” by Jean Meeus](http://www.willbell.com/math/mc1.htm). This book is recommended by the Astronomical Applications Department of the U.S. Naval Observatory and the Earth System Research Laboratory of the National Oceanic and Atmospheric Administration.

Implementations of Adhan in other languages can be found in the [Adhan](https://github.com/batoulapps/Adhan) repo.

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/iamriajul/adhan-dart/issues
