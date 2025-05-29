# TY NVR App Support

Thank you for using the TY NVR app. This app allows users to view live streams and recorded events from Frigate, an open-source NVR system.

## 📘 Help & Setup

To connect your app with Frigate, follow our setup instructions:  
👉 [Frigate Documentation](https://docs.frigate.video/)

## 🔥 Firebase Setup Guide

To enable push notifications, you need to create a Firebase project and obtain the configuration keys.

1. Go to the [Firebase Console](https://console.firebase.google.com/) and click "Add project".
2. Name your project (e.g., "TY NVR") and click "Continue". You can enable or disable Google Analytics as needed.
3. After the project is created, go to the project dashboard, click the gear icon ⚙️ → **Project settings**.
4. Under the "General" tab, find and copy the following details:
   - **Project ID**
   - **Web API Key**
   - **App ID** (e.g., `1:1234567890:web:abcdef123456`)
   - **Messaging Sender ID**
5. To enable Cloud Messaging (push notifications), go to the "Build" section → **Cloud Messaging** and ensure it's enabled.
6. (Optional) For mobile app push support (iOS/Android), configure your FCM keys and certificates under "Cloud Messaging" in project settings.

## 💬 Contact Us

If you need assistance, please contact us at:  
**Email:** [support@147lab.com](mailto:support@147lab.com)

## 🔗 About Frigate

Frigate is an open-source NVR project developed by the community. Learn more at:  
[https://frigate.video](https://frigate.video)

---

*This app is a third-party client and is not officially affiliated with the Frigate project.*
