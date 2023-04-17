Build an android app without Android Studio (2023-04-17, macOS)

```
# https://android.googlesource.com/platform/packages/apps/Benchmark/ 
# https://zenn.dev/110416/articles/0945183fe53740 
# download java
mkdir -p ~/android
cd ~/android
wget https://download.oracle.com/java/17/latest/jdk-17_macos-x64_bin.dmg
open jdk-17_macos-x64_bin.dmg
java --version | grep 'java 17' && echo "OK"
# setup cmdline-tools
mkdir -p ~/android/cmdline-tools
cd ~/android/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-mac-9477386_latest.zip
unzip commandlinetools-mac-9477386_latest.zip
mv cmdline-tools latest
export PATH="$HOME/android/cmdline-tools/latest/bin:$PATH"
sdkmanager --version | grep '9.0' && echo "OK"
# setup sdk
yes | sdkmanager --licenses
sdkmanager --install platform-tools
sdkmanager --install emulator
sdkmanager --install "system-images;android-30;default;x86_64"
sdkmanager --install "build-tools;30.0.3"  "platforms;android-30"
# create an emulator
avdmanager create avd --name android30_x86_64 --device "pixel"  --package "system-images;android-30;default;x86_64"
# start emulator
~/android/emulator/emulator -avd android30_x86_64
# it will take 36s or more
adb devices | grep emulator-5554
adb shell


# build
cd ~/repo/Benchmark/app/src/main
export PLATFORM=`readlink -f ~/android/platforms/android-30/android.jar`
mkdir -p src/com/android/benchmark/
~/android/build-tools/30.0.3/aapt package -f -m -J src -M AndroidManifest.xml -S res -I ~/android/platforms/android-30/android.jar
javac -d obj --release 7 -classpath ~/android/platforms/android-30/android.jar src/com/android/benchmark/*.java
jar tf ~/android/platforms/android-30/android.jar | grep android | grep Bundle
~/android/build-tools/30.0.3/dx --dex --output=classes.dex obj
mkdir bin
~/android/build-tools/30.0.3/aapt package -f -m -F bin/hello.unaligned.apk -M AndroidManifest.xml -S res -I ~/android/platforms/android-30/android.jar
~/android/build-tools/30.0.3/aapt add bin/hello.unaligned.apk classes.dex
keytool -genkeypair -validity 365 -keystore mykey.keystore -keyalg RSA -keysize 2048
~/android/build-tools/30.0.3/apksigner sign --ks mykey.keystore bin/hello.unaligned.apk
~/android/build-tools/30.0.3/zipalign -f 4 bin/hello.unaligned.apk bin/hello.apk
adb install bin/hello.unaligned.apk
adb shell am start -n com.android.benchmark/.MainActivity

```
