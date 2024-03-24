![](assets/image1.png){width="6.0in"
height="1.4097222222222223in"}

> ![](assets/image2.png){width="3.75in"
> height="3.75in"}
>
> **Synopsis (!)**
>
> Rocket\
> 24 March 2024\
> Prepared By: 0xFF\
> Challenge Author(s): 0xFF Difficulty: Hard

+-----------------------------------+-----------------------------------+
| •                                 | > Firstly, the user should grab   |
|                                   | > the APK file from an AAB        |
|                                   | > format. AABs are a type of      |
|                                   | > Android file used to publish    |
|                                   | > and distribute Android apps.    |
|                                   | > They are                        |
+===================================+===================================+
+-----------------------------------+-----------------------------------+

> now the standard file format used to publish Android apps on the
> Google Play Store. More info
> here(https://developer.android.com/guide/app-bundle/app-bundle-format).
> Once the user gets the APK file, the first step is to decompile it.
> They will soon realize that the login screen is not functional yet, so
> the next step is to bypass the Login Activity using the Intent\
> Redirection vulnerability. This vulnerability will navigate the user
> to the second Rockets Activity which contains a list of rockets. Each
> rocket click will navigate the user to a details screen. After
> analyzing the Rocket Details Activity, the user should modify the APK
> to get the secret flag.
>
> **Description (!)**
>
> • This challenge is about Intent Redirection vulnerability in Android
> apps. An

intent redirection occurs when an attacker can partly or fully control
the

contents of an intent used to launch a new component in the context of a

> vulnerable app. In other words, using this vulnerability, components
> of
>
> vulnerable apps can be accessed by third-party apps.
>
> **Skills Required (!)**
>
> • Android
>
> • Researching Skills
>
> • Kotlin, Smali
>
> • Know how to use tools such as apktool, jadx, keytool and jarsigner
>
> • Android Studio, VS Code
>
> **Skills Learned (!)**
>
> • Learn about the AAB format.
>
> • Learn how Intent Redirection vulnerability works.
>
> • Learn how to decompile an APK.
>
> • Learn how to analyze the decompiled APK.
>
> • Learn how to modify an APK.
>
> • Learn how to rebuild the modified APK.
>
> • Learn how to generate a new keystore.
>
> • Learn how to sign an APK.
>
> **Enumeration (!)**
>
> **Analyzing the source code (\*)**
>
> Once the user unzips the file, they will find the app in AAB format
> (Android app
>
> bundle). AABs are a type of Android file used to publish and
> distribute Android
>
> apps. They are now the standard file format used to publish Android
> apps on the
>
> Google Play Store. More info
> here(https://developer.android.com/guide/app-
>
> bundle/app-bundle-format)
>
> Therefore, the first step is to grab the APK from the AAB file with
> the Bundle tool
>
> (You need to download the Bundle tool if it does not exist):
>
> ![](assets/image3.png){width="6.405555555555556in"
> height="0.23472112860892388in"}
>
> Once the rockets.apks file is generated, we need to **change the
> file's extension**
>
> **from rockets.apks to rockets.zip:**
>
> ![](assets/image4.png){width="5.677777777777778in"
> height="0.3333333333333333in"}
>
> Unzip the rockets.zip file and the APK we need is the
> **universal.apk** inside the rockets folder.
>
> We can now install the APK to navigate in the app. The only screen we
> can see is the login which needs credentials, so there is nothing else
> to do here.
>
> ![](assets/image5.png){width="2.2916666666666665in"
> height="4.074998906386702in"}
>
> Let's dive into the code!
>
> First, we should decompile the APK to get a better picture (Download
> the apktool if it does not exist):
>
> ![](assets/image6.png){width="5.030555555555556in"
> height="0.25in"}
>
> Use the jadx tool to analyze the code:
>
> ![](assets/image7.png){width="3.4472222222222224in"
> height="0.22916666666666666in"}
>
> Once the jadx tool opens, we select the project that we decompiled:
>
> ![](assets/image8.png){width="4.754166666666666in"
> height="2.811111111111111in"}
>
> We can see that there are 3 activities. (LoginActivity,
> RocketsActivity, and RocketDetailActivity):

![](assets/image9.png){width="6.0in"
height="2.0569433508311463in"}

> Let's look inside the Login Activity:

![](assets/image10.png){width="6.0in"
height="2.9972222222222222in"}

> There is a button that does some work when clicked. It looks like it
> calls a function **d2.a.** If we navigate there, we can see an
> interesting piece of code that checks a condition to start an
> activity.

![](assets/image11.png){width="6.0in"
height="2.997221128608924in"}

> Potentially, if we modify the condition, we could break into the app.
>
> However, if we are more careful, we can see that the last line inside
> the if condition does nothing. It just tries to start an activity with
> a new Intent empty. This won\'t work. Therefore, even if we could find
> the credentials, we wouldn't be able to get past the login wall
> because this intent does nothing.
>
> So, let's take a look at the Rockets List Activity:

![](assets/image12.png){width="6.0in"
height="2.9972222222222222in"}

> It seems that sets a recycler view but still does not contain
> something interesting.
>
> Lastly, the Details List Activity:

![](assets/image13.png){width="6.0in"
height="2.9972222222222222in"}

> There is a piece of code at the end of the activity that defines a
> textView with more details, but before defining checks a condition.
> There is also a hardcoded string
>
> there called "**secret**". We could try to work with this case but
> still, we do not have access to this activity to modify the code and
> check the results, as we are blocked by the login wall.
>
> **Solution (!)**
>
> **Finding the vulnerability (\*)**
>
> Let's take another approach and take a look at the AndroidManifest.xml
> file. Indeed, there are 3 activities declared there. If we look
> carefully there is an attribute **android:exported** declared on each
> activity. The android:exported attribute sets whether a
> component(activity, service, broadcast reaceiver, etc.) can be
> launched by components of other applications. If true, any app can
> access the activity and launch it by its exact class name. If false,
> only components of the same application,\
> applications with the same user ID, or privileged system components
> can launch the\
> ).

![](assets/image14.png){width="6.0in"
height="1.5166655730533682in"}

> The android:exported attribute seems to be declared as true in the
> Login Activity. This is justified as login activity is the first
> screen when the app launches. However, android:exported attribute has
> also been declared as true in Rockets Activity. **This means that we
> could bypass the login screen if we make an intent to Rockets Activity
> directly.**
>
> **Exploitation (!)**
>
> Therefore, the next step is to **create an attacker app that will send
> an intent to the vulnerable app.**
>
> **Creating the attacker app (\*)**
>
> First, we open the Android studio and create a new project.
>
> The attacker's app UI will contain only one activity and a single
> button in it. Upon clicking the button, the app will send an Intent to
> the vulnerable app and try to open the Rockets activity declaring the
> right path.
>
> The main activity of the attacker's app:

![](assets/image15.png){width="6.0in"
height="3.5722222222222224in"}

> •We declare a constant variable as the package name we target to make
> the intent.
>
> •We set up an onClickListener for the button.
>
> •Inside the listener, we create the intent and set the component of
> the vulnerable path.
>
> •Start the activity-component.
>
> The main activity XML file represents the UI:
>
> ![](assets/image16.png){width="5.206944444444445in"
> height="2.554165573053368in"}
>
> We only added a button there. That's all! Let's run the attacker's app
> and click the button.
>
> The Rockets Activity opens, containing a list of rockets:
>
> ![](assets/image17.png){width="2.066666666666667in"
> height="3.672222222222222in"}
>
> Even if the user clicks on each rocket, they only are navigated to a
> details screen:
>
> ![](assets/image18.png){width="2.0569444444444445in"
> height="3.6555555555555554in"}
>
> Nothing interesting on either screen. But if we analyze the details
> screen a bit, we can see that the UI consisted of 3 elements. An
> imageView and 2 textViews.
>
> Previously we found in the RocketDetailActivity a case that defined a
> textView3 with more details. **Thus, there is one more textView which
> is hidden.**
>
> **Getting the flag (!)**
>
> Let's work with the case we found in the Rocket Detail Activity
> previously:
>
> ![](assets/image19.png){width="5.15in"
> height="2.573611111111111in"}
>
> We can try to modify this if condition or boolean to see what value is
> filled in textView3.
>
> First, we open the VS code to see the SMALI classes:

![](assets/image20.png){width="6.0in"
height="0.5944444444444444in"}

> When the VS code opens, we navigate to the RocketDetailScreen:

![](assets/image21.png){width="6.0in"
height="2.9972222222222222in"}

> We can either modify the Boolean flag:

![](assets/image22.png){width="6.0in"
height="1.1902777777777778in"}

> Or the if condition:

![](assets/image23.png){width="6.0in"
height="1.0180555555555555in"}

> We worked with the second case and changed the condition to **if-nez**
> to achieve the opposite result and the textView3 to be defined.
>
> Now we need to save the change and rebuild the APK:
>
> ![](assets/image24.png){width="5.375in"
> height="1.6124989063867017in"}
>
> If any error occurs as above, a new version of apktool should be
> installed and run the command:

![](assets/image25.png){width="6.0in"
height="0.22777777777777777in"}

> The next step is to sign this APK.\
> First, we create a new keystore:

![](assets/image26.png){width="6.0in"
height="2.1638877952755906in"}

> Then, we sign the modified APK:

![](assets/image27.png){width="6.0in"
height="0.4in"}

> We enter the passphrase we defined on the keystore's creation.
>
> Finally, after deleting the previous APK from the emulator or device,
> we install the modified APK.
>
> Surprisingly, when we tried to install there is another error:
>
> ![](assets/image28.png){width="3.5in"
> height="1.1138877952755906in"}
>
> This happens because the challenge flag was secured with a native
> library. More info abou\
> here)\
> Therefore, to be able to install the application we should modify an
> attribute inside the AndroidManifest.xml file.
>
> We open again the decompiled folder called universal and open the
> AndroidManifest.xml file:

![](assets/image29.png){width="6.0in"
height="2.959722222222222in"}

> All we have to do is to change android:extractNativeLibs from false to
> true.
>
> Save the change, build, and sign the APK as we did previously.
>
> Install the new APK and run the attacker's app again.
>
> When we click the button the Rockets Activity opens.
>
> Navigate to the details screen, by clicking a rocket.
>
> Get the flag:
>
> ![](assets/image30.png){width="2.102777777777778in"
> height="3.738888888888889in"}
