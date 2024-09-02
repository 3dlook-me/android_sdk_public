
# 3DLOOK Camera UI Android SDK  
## Integration of the Package

### 1. Generate Personal Access Token in Your Github Account ([official documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens))

Follow the steps below to generate a personal access token:

1. Go to **Settings** -> **Developer Settings** -> **Personal Access Tokens** -> **Generate new token**.
2. Make sure to select the following scope: `read:packages`.
3. Click **Generate token**.

<span style="color:red">**Important:**</span> After generating the token, **make sure to copy your new personal access token**. You cannot see it again! The only option is to generate a new key.

### 2. Store Your GitHub Personal Access Token Details
1. Create a `github.properties` file within the root of your Android project.
2. If this is a public repository, make sure you add this file to `.gitignore` to keep the token private.
3. Add the following properties:
	```properties
	gpr.usr=GITHUB_USERID
	gpr.key=PERSONAL_ACCESS_TOKEN
	```
4. Replace `GITHUB_USERID` with your personal or organization GitHub User ID and replace `PERSONAL_ACCESS_TOKEN` with the token generated in [step 1](#1-generate-personal-access-token-in-your-github-account-offial-documentation)  
**Note**: Alternatively, you can add the `GPR_USER` and `GPR_API_KEY` values to your environment variables on your local machine or build server to avoid creating a GitHub properties file.

### 3. Configure Build Files

Depending on your build configuration language, pick the appropriate step below (3.1 or 3.2).

#### 3.1. Kotlin DSL

1. Update `settings.gradle.kts` inside the project with the GitHub repository path and credentials:
	```kotlin
	import java.io.FileInputStream
	import java.util.Properties

	val githubProperties = Properties().apply {
		load(FileInputStream(File(rootDir, "github.properties")))
	}
	```
2. Add the following repository:
	```kotlin
	dependencyResolutionManagement {
		...
		repositories {
			...
			maven {
				name = "GitHubPackages"
				url = uri("https://maven.pkg.github.com/3dlook-me/android_sdk_public")

				credentials {
						username = githubProperties["gpr.usr"] as String? ?: System.getenv("GPR_USER")
						password = githubProperties["gpr.key"] as String? ?: System.getenv("GPR_API_KEY")
				}
			}
		}
	}
	```
3. Add the camera dependency in `build.gradle.kts` inside the app module:
	```kotlin
	dependencies {
		implementation("com.look.camera.sdk:look-camera-sdk:0.0.2")
	}
	```

#### 3.2. Groovy DSL

1. Update `settings.gradle` inside the project with the GitHub repository path and credentials:
	```groovy
	def githubProperties = new Properties()
	githubProperties.load(new FileInputStream(new File(rootDir, "github.properties")))
	```
2. Add the following repository:
	```groovy
	dependencyResolutionManagement {
		...
		repositories {
			...
			maven {
				name = "GitHubPackages"
				url = uri("https://maven.pkg.github.com/3dlook-me/android_sdk_public")
				credentials {
						username = githubProperties.getProperty("gpr.usr") ?: System.getenv("GPR_USER")
						password = githubProperties.getProperty("gpr.key") ?: System.getenv("GPR_API_KEY")
				}
			}
		}
	}
	```
3. Add the camera dependency in `build.gradle` inside the app module:
	```groovy
	dependencies {
		...
		implementation("com.look.camera.sdk:look-camera-sdk:0.0.2")
	```

### 4. Add Camera Permissions to the Application

Add the following permissions to the `AndroidManifest.xml` inside the app module:
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
	<uses-permission android:name="android.permission.CAMERA"/>
	<uses-feature android:name="android.hardware.camera" android:required="false" />
```

### 5. Sync Your Android Gradle Project

Make sure to sync your Android Gradle project to apply the changes.

### 6. Example of the Usage

1. Import the library:
	```kotlin
	import com.look.camera.sdk.SdkActivity
	import com.look.camera.sdk.data.LaunchOption
	```
2. Create a property to store the initial launch option for the library:
	```kotlin
	var launchOption = LaunchOption.START_FROM_TUTORIAL
	```
	Available options:
	* **START_FROM_TUTORIAL** - Will start from the tutorials
	* **FRONT_ONLY** - Only for retaking of the front photo
	* **SIDE_ONLY** - Only for retaking of the side photo
	* **FRONT_AND_SIDE_ONLY** - For retaking of both photos
3. Create a property to handle the callbacks from the library:
	```kotlin
	private val launcher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
		if (result.resultCode == RESULT_OK) {
			 val frontPhotoUri = SdkActivity.getFrontPhotoUri(result.data)
			 val sidePhotoUri = SdkActivity.getSidePhotoUri(result.data)
			 //Example of retaking photos
			 //After validation of photos on the server - we check if we want to launch SDK again
			 // we can pick any of the LaunchOption values: FRONT_ONLY, SIDE_ONLY, FRONT_AND_SIDE_ONLY
			 launchOption = LaunchOption.FRONT_ONLY

			 Log.d("MainActivity CALLBACK", "Front photo URI: $frontPhotoUri, Side photo URI: $sidePhotoUri")
			 // Handle the URIs
		}
	}
	```
	`frontPhotoUri` and `sidePhotoUri` represent the links to the photos stored in a cache folder. (For example, after verifying both photos, you may need to pass a new value for `launchOption` into `SdkActivity` to retake the incorrect photo/photos).

4. Launch the activity (an example below for the Compose):
	```kotlin
		val intent = SdkActivity.start(this@MainActivity,
			 launchOption)
		launcher.launch(intent)
	```
