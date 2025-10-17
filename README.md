# Native Instruments software uses PORT 5562 ON TCP (NTKDaemon)

# Run CMD as Administrador and write:

netstat -ano | findstr :5563
taskkill /PID <PID> /F

C:\Users\andre\AppData\Local\Android\Sdk\platform-tools\adb

> adb kill-server
> adb start-server
> adb devices

# Test on android

npm run build
npx cap run android

# Build on android

npm run build
npx cap open android
top menu > build > Generate bundle/apk > apk

# Update icon and splash screen

npx capacitor-assets generate
