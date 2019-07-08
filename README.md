# CircleCI Instrumentation Tests for Android
An example configuration which shows how you can run Instrumentation Tests for Android on Circle CI without a third party service.

## Getting Started

You need to initialize CircleCI and fastlane in order to use CircleCI Video Recording. You can setup CircleCI by following [this documentation](https://circleci.com/docs/2.0/getting-started/) and fastlane with [this one](https://docs.fastlane.tools/getting-started/ios/setup/). MacOS environment is needed in order to run Android instrumentation tests.

## Usage

Copy .circleci and Fastlane folders from this repository to your project. After this, with every commit you make to Github, a commit workflow which runs test will start on CircleCI.

## How It Works
The commit triggers the `ui_test` job on CircleCI, the job first set up the Android development environment and starts the emulator then runs `ui_test` lane from Fastlane.

## Setting Up Android Environment in CircleCI Config
In order to run tests we need an active Android device emulator and we can set it up from `.circleci/config.yml` file. We need to execute the following prompts in our CircleCI job which is named `ui_test` in the example.

### Setting Environment Variables
```yml
- run:
    name: Setup environment variables
    command: |
      echo 'export PATH="$PATH:/usr/local/opt/node@8/bin:${HOME}/.yarn/bin:${HOME}/${CIRCLE_PROJECT_REPONAME}/node_modules/.bin:/usr/local/share/android-sdk/tools/bin"' >> $BASH_ENV
      echo 'export ANDROID_HOME="/usr/local/share/android-sdk"' >> $BASH_ENV
      echo 'export ANDROID_SDK_HOME="/usr/local/share/android-sdk"' >> $BASH_ENV
      echo 'export ANDROID_SDK_ROOT="/usr/local/share/android-sdk"' >> $BASH_ENV
      echo 'export QEMU_AUDIO_DRV=none' >> $BASH_ENV
      echo 'export JAVA_HOME=/Library/Java/Home' >> $BASH_ENV
```
### Installing the SDK and the Emulator
```yml
- run:
    name: Install Android sdk
    command: |
      HOMEBREW_NO_AUTO_UPDATE=1 brew tap homebrew/cask
      HOMEBREW_NO_AUTO_UPDATE=1 brew cask install android-sdk
- run:
    name: Install emulator
    command: (yes | sdkmanager "platform-tools" "platforms;android-26" "extras;intel;Hardware_Accelerated_Execution_Manager" "build tools;26.0.0" "system-images;android-26;default;x86" "emulator" --verbose) || true
```
### Starting the Emulator and Running Tests
```yml
- run: avdmanager create avd -n Nexus_5X_API_26_x86 -k "system-images;android-26;default;x86" -d "Nexus 5X"
      - run: osascript ./fastlane/recording_related/dismiss_warning.scpt # Dismisses the same named computer error
      - run:
          name: Run emulator in background
          command: /usr/local/share/android-sdk/tools/emulator @Nexus_5X_API_26_x86 -skin 1080x2066 -memory 2048 -noaudio
          background: true
- run:
      name: Run Tests
      command: bundle exec fastlane ui_test
      no_output_timeout: 30m
```

The `bundle exec fastlane ui_test` command starts the relevant fastlane lane.
## Running Tests from Fastlane
```ruby
desc "Do UI Tests on CircleCI"
lane :ui_test do
    # Run all Tests
    sh('cd .. && ./gradlew app:connectedAndroidTest')
    # Run spesific Tests
    # sh('cd .. && ./gradlew app:connectedAndroidTest -Pandroid.testInstrumentationRunnerArguments.package=<INSERT_YOUR_TEST_PACKAGE_HERE(com.app.myapp.emulatortests)>')
end
```

## Author

[Connected2.me](http://connected2.me) / <a href="mailto:dogudeniz.ugur@gmail.com">Dogu Deniz Ugur</a> <a href="https://github.com/DoguD">@DoguD</a>

## License

CircleCI Instrumentation Tests for Android is available under the MIT license. See the LICENSE file for more info.
