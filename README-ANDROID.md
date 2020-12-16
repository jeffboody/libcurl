About
=====

This repository contains a mirror of the libcurl library.
For more information see the https://curl.se/ website.

The main purpose for this mirror is to provide header files
which can be used with Android native dependencies and
Android Archives (AAR). The problem with AAR is that the
curl.h header is not visible to subdirectories/submodules.
To work around this limitation you can include the curl.h
header directly from libcurl rather than getting the
definition from the AAR.

Note that there is a Google issue tracking the linking of
C++ libraries which are not actually used by Curl or
OpenSSL.

	https://github.com/google/prefab/issues/134

build.gradle
============

	diff --git a/app/build.gradle b/app/build.gradle
	index 894fb79..252273e 100644
	--- a/app/build.gradle
	+++ b/app/build.gradle
	@@ -11,6 +11,7 @@ android {
	         testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
	         externalNativeBuild {
	             cmake {
	+                arguments "-DANDROID_STL=c++_shared"
	                 cppFlags ""
	             }
	         }
	@@ -21,6 +22,9 @@ android {
	             proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
	         }
	     }
	+    buildFeatures {
	+        prefab true
	+    }
	     externalNativeBuild {
	         cmake {
	             path "src/main/cpp/CMakeLists.txt"
	@@ -32,6 +36,8 @@ dependencies {
	     implementation fileTree(dir: 'libs', include: ['*.jar'])
	     implementation 'androidx.appcompat:appcompat:1.0.0'
	     implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
	+    implementation 'com.android.ndk.thirdparty:curl:7.79.1-beta-1'
	+    implementation 'com.android.ndk.thirdparty:openssl:1.1.1l-beta-1'
	     testImplementation 'junit:junit:4.12'
	     androidTestImplementation 'androidx.test.ext:junit:1.1.1'
	     androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'

CMakeLists.txt
==============

	diff --git a/app/src/main/cpp/CMakeLists.txt b/app/src/main/cpp/CMakeLists.txt
	index 929ec09..ffc76d5 100644
	--- a/app/src/main/cpp/CMakeLists.txt
	+++ b/app/src/main/cpp/CMakeLists.txt
	@@ -12,6 +12,9 @@ if(TEXGZ_USE_JPEG)
	         myjpeg)
	 endif()

	+# AAR dependencies (Android Archive)
	+find_package(curl REQUIRED CONFIG)
	+
	 # ANativeActivity interface
	 set(CMAKE_SHARED_LINKER_FLAGS
	     "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")
	@@ -62,6 +65,9 @@ target_link_libraries(APP

	+                      # AAR libraries
	+                      curl::curl
	+
	                       # NDK libraries
	                       android
	                       log

References
==========

Android Native Dependencies

	https://developer.android.com/studio/build/native-dependencies?buildsystem=cmake
	https://android-developers.googleblog.com/2020/02/native-dependencies-in-android-studio-40.html
	https://github.com/android/ndk-samples/tree/master/prefab/curl-ssl
