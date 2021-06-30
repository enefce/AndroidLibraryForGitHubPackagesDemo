
[![Build Status](https://github.com/enefce/AndroidLibraryForGitHubPackagesDemo/workflows/Android%20CI/badge.svg)](https://github.com/enefce/AndroidLibraryForGitHubPackagesDemo/actions)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fenefce%2FAndroidLibraryForGitHubPackagesDemo.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fenefce%2FAndroidLibraryForGitHubPackagesDemo?ref=badge_shield)
[![Library Version](https://img.shields.io/badge/LibraryVersion-v1.0.2-brightgreen)](https://github.com/enefce/AndroidLibraryForGitHubPackagesDemo/packages/50498)

![License](https://img.shields.io/github/license/enefce/AndroidLibraryForGitHubPackagesDemo?color=2fc544)


# Sample Android Library Publishing to GitHub Package Registry

  This Android project showcases the steps to publish and consume Android Libraries on the GitHub Package Registry.
   It is made up of 2 modules 
  1.  ##### sampleAndroidLib
   - Android library module with basic functionality
   - Publishes the generated library file onto the GitHub Package Registry
   - The build.gradle file inside this module has the code (plugin, tasks and authentication) related to publishing the library
  2.  #####  app
   - Sample Android application module with the build.gradle file that shows the code for consuming an Android library from GitHub Package Registry.
 
------------
## Publish Android library to GitHub Package Registry

### Step 1 : Generate a Personal Access Token for GitHub
- Inside you GitHub account:
	- Settings -> Developer Settings -> Personal Access Tokens -> Generate new token
	- Make sure you select the following scopes (" write:packages", " read:packages") and Generate a token
	- After Generating make sure to copy your new personal access token. You won’t be able to see it again!

### Step 2: Store your GitHub - Personal Access Token details
- Create a **github.properties** file within your root Android project
- In case of a public repository make sure you  add this file to .gitignore for keep the token private
	- Add properties **gpr.usr**=*GITHUB_USERID* and **gpr.key**=*PERSONAL_ACCESS_TOKEN*
	- Replace GITHUB_USERID with personal / organisation Github User ID and PERSONAL_ACCESS_TOKEN with the token generated in **#Step 1**
	
> Alternatively you can also add the **GPR_USER** and **GPR_API_KEY** values to your environment variables on you local machine or build server to avoid creating a github properties file

### Step 3 : Update build.gradle inside the library module
- Add the following code to **build.gradle** inside the library module
```gradle
apply plugin: 'maven-publish'
```
```gradle
def githubProperties = new Properties()
githubProperties.load(new FileInputStream(rootProject.file("github.properties")))  
```
```gradle
def getVersionName = { ->
    return "1.0.2"  // Replace with version Name
}
```
```gradle
def getArtificatId = { ->
    return "sampleAndroidLib" // Replace with library name ID
}
```
```gradle
publishing {
    publications {
        bar(MavenPublication) {
            groupId 'com.enefce.libraries' // Replace with group ID
            artifactId getArtificatId()
            version getVersionName()
            artifact("$buildDir/outputs/aar/${getArtificatId()}-release.aar")
        }
    }

    repositories {
        maven {
            name = "GitHubPackages"
            /** Configure path of your package repository on Github
             *  Replace GITHUB_USERID with your/organisation Github userID and REPOSITORY with the repository name on GitHub
             */
            url = uri("https://maven.pkg.github.com/GITHUB_USERID/REPOSITORY")

            credentials {
                /**Create github.properties in root project folder file with gpr.usr=GITHUB_USER_ID  & gpr.key=PERSONAL_ACCESS_TOKEN**/
                username = githubProperties['gpr.usr'] ?: System.getenv("GPR_USER")
                password = githubProperties['gpr.key'] ?: System.getenv("GPR_API_KEY")
            }
        }
    }
}
```
### Step 4 : Publish the Android Library onto GitHub Package Registry
> Make sure to build and run the tasks to generate the library files inside ***build/outputs/aar/*** before proceeding to publish the library.


- Execute the ****Publish**** gradle task which is inside your library module
  
```console
$ gradle publish
```
- Once the task is successful you should be able to see the Package under the **Packages** tab of the GitHub Account
- In case of a failure run the task with *--stacktrace*, *--info* or *--debug* to check the logs for detailed information about the causes.
	
	

------------
## Using a library from the GitHub Package Registry
> Currently the GitHub Package Registry requires us to Authenticate to download an Android Library (Public or Private) hosted on the GitHub Package Registry. This might change for future releases

> Steps 1 and 2 can be skipped if already followed while publishing a library

### Step 1 : Generate a Personal Access Token for GitHub
- Inside you GitHub account:
	- Settings -> Developer Settings -> Personal Access Tokens -> Generate new token
	- Make sure you select the following scopes ("read:packages") and Generate a token
	- After Generating make sure to copy your new personal access token. You won’t be able to see it again!

### Step 2: Store your GitHub - Personal Access Token details
- Create a **github.properties** file within your root Android project
- In case of a public repository make sure you  add this file to .gitignore for keep the token private
	- Add properties **gpr.usr**=*GITHUB_USERID* and **gpr.key**=*PERSONAL_ACCESS_TOKEN*
	- Replace GITHUB_USERID with personal / organisation Github User ID and PERSONAL_ACCESS_TOKEN with the token generated in **#Step 1**
	
> Alternatively you can also add the **GPR_USER** and **GPR_API_KEY** values to your environment variables on you local machine or build server to avoid creating a github properties file

### Step 3 : Update build.gradle inside the application module 
- Add the following code to **build.gradle** inside the application module that will be using the library published on GitHub Packages Repository
```gradle
def githubProperties = new Properties()
githubProperties.load(new FileInputStream(rootProject.file("github.properties")))  
```
```gradle
    repositories {
        maven {
            name = "GitHubPackages"
            /*  Configure path to the library hosted on GitHub Package Registry
             *  Replace UserID with package owner userID and REPOSITORY with the repository name
             *  e.g. "https://maven.pkg.github.com/enefce/AndroidLibraryForGitHubPackagesDemo"
             */
            url = uri("https://maven.pkg.github.com/UserID/REPOSITORY")

            credentials {
                username = githubProperties['gpr.usr'] ?: System.getenv("GPR_USER")
                password = githubProperties['gpr.key'] ?: System.getenv("GPR_API_KEY")
            }
        }
    }
```

- inside dependencies of the build.gradle of app module, use the following code
```gradle
dependencies {
    //consume library, e.q. 'com.enefce.libraries.sampleAndroidLib:sampleandroidlib:1.0.5'
    implementation '<groupId>:<artifactId>:<version>'
	...
}
```



## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fenefce%2FAndroidLibraryForGitHubPackagesDemo.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fenefce%2FAndroidLibraryForGitHubPackagesDemo?ref=badge_large)
