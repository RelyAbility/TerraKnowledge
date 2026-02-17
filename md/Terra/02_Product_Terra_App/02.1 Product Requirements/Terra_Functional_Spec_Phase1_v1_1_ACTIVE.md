# Terra -- Functional Specification (Phase 1 Mobile App)

## 1. Platform Architecture

The application will be built using React Native with a shared codebase
for iOS and Android. Platform-specific modules may be implemented where
required to support offline functionality, push notifications, and
OS-level behaviour.

## 2. Core User Inputs

• Plot boundary (polygon draw or upload)

• Location selection within 10 km flagship zone

• Notification preferences (Monthly Brief, Mission reminders, Alerts)

## 3. System Processing

• Retrieve regional ecosystem mapping intersecting plot

• Generate monthly NDVI composite

• Calculate structural complexity proxy

• Overlay disturbance and connectivity layers

• Match ecosystem to native vegetation preference library

• Cache last Monthly Brief locally for offline viewing

• Queue offline mission updates for later sync

## 4. System Outputs

• Monthly Ecological Intelligence Brief (push-triggered)

• Vegetation health indicator and trend

• Structural complexity indicator

• Mission recommendations (structural interventions)

• Disturbance alerts (optional notifications)

## 5. Offline Requirements

• Local storage of most recent Monthly Brief

• Local storage of mission checklist and guidance

• Ability to record notes/photos offline

• Background sync when connectivity restored

• Limited offline map tile caching (user-initiated area download)

## 6. Push Notification Requirements

• Monthly Brief release notification

• Optional mission reminder notifications

• Optional disturbance alert notifications

• Platform-specific permission handling (iOS/Android)

## 7. Edge Cases

• User declines notification permissions

• Prolonged offline usage

• Cloud-heavy month with limited satellite data

• Small plots near pixel resolution threshold
