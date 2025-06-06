# buenro-tests

## QA environment setup:

1. Clone repository with automated tests to your local machine:

- `git clone https://github.com/yurymaroz/buenro-tests.git`

2. Install Maestro to your local machine using step-by-step guide below:
   https://docs.maestro.dev/getting-started/installing-maestro
3. Open Visual Studio Code (https://code.visualstudio.com/download) --> File --> Open Folder --> Select cloned repository from your machine.
4. Enable terminal window under your opened project in VSC i.e. you should be in terminal window under project folder.
   Visual Studio Code --> View --> Enable Terminal.
5. Install Chrome browser: https://www.google.com/chrome.
6. Install Java: https://www.java.com/en/download/manual.jsp
7. Install Android Studio (including android-sdk + virtual device manager): https://developer.android.com/studio
8. Set ANDROID_HOME variable on your machine
9. Add platform-tools to System PATH Environment Variables:
   C:\Users\<YourName>\AppData\Local\Android\Sdk\platform-tools
10. Start Android Studio -> Virtual Device Manager.
11. Start any preferable device/emulator.
12. Clone repository with flutter app to your local machine:

- `git clone https://github.com/Buenro/hotel-search.git`

13. Go to directory with cloned repo e.g

- `cd hotel-search`

14. Install Dependencies

- `flutter pub get`

15. Create SerpApi Api Key (https://serpapi.com/dashboard).
16. Create a .env file in the root directory and add your SerpAPI key:

SERPAPI_API_KEY=<YOUR_API_KEY>

17. Run the Dart Code Generator and Generate necessary files using the build_runner package:

- `flutter pub run build_runner build --delete-conflicting-outputs`

18. Generate apk build for Android:

- `flutter build apk --debug`

19. Install apk build to connected Android device/emulator from step 10:

- `adb install -r build/app/outputs/flutter-apk/app-debug.apk`

## Command to run tests via CLI

- `maestro test .maestro --debug-output ./output`

Note: There is maestro related limitation that tests are not running when Maestro Studio is launched.

## Dependencies and Tools

| Tool      | Use Case                | Version |
| --------- | ----------------------- | ------- |
| `maestro` | End-to-end flow testing | 1.40.3  |

## Continuous Integration in GitHub Actions

1.  Maestro cloud is used for CI, so please create account here: https://signin.maestro.dev/sign-up
2.  Login to Maestro cloud from previous step 1 --> Api Key --> Generate new api key and save it
3.  Save your Project ID that can be found in your logged in url for Maestro cloud:
    https://app.maestro.dev/project/{YourProjectID}/dashboard
4.  Go to your repository in GitHub and open settings:
    https://github.com/yurymaroz/buenro-tests/settings
5.  Secrets and variables --> Actions --> Add the following repository secrets:
    MAESTRO_API_KEY (from step 2)
    MAESTRO_PROJECT_ID (from step 3)
    SERPAPI_API_KEY (from step 15 of previous section README)
6.  Find sample working config to setup tests execution in GitHub Actions:
    .github -> workflows -> maestro.yml
7.  Please note at this moment there is a rule to run tests after any push to main branch:
    `on:
push:
    branches:
        - main`
8.  There is also way to run current workflow manually. The part below of config is responsible for it:
    `on:
workflow_dispatch:`
9.  There is also another option to setup schedule for workflow. Then config should be updated with it (example for every 4 hours):
    `on:
    schedule:
    - cron: '0 _/4 _ \* \*'`
10. How to include test execution as part of a build workflow:
    Go to `jobs` section of current config file and add more test execution like already setup for `tests` job.
11. Where to collect logs, screenshots, or other test artifacts:
    Go to your project dashboard of Maestro Cloud to see current test execution output (https://app.maestro.dev/project/{YourProjectID}/dashboard)
12. How to configure Built-in GitHub notifications:
    These are automatically triggered by GitHub when workflow runs fail, succeed, or are cancelled—based on your notification settings.
    Steps:

        - Go to Settings > Notifications: https://github.com/settings/notifications
        - Customize your preferences: Email, Web (GitHub UI)
        - Set the "Actions" event to the desired frequency (e.g., "Only failures").

13. How to configure manual notification via Actions (e.g. Slack and etc.)
    Use an action step in your workflow to send messages when jobs succeed or fail. For example:

    - name: Notify Slack on Failure
      if: failure()
      uses: rtCamp/action-slack-notify@v2
      env:
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
      SLACK_COLOR: '{{color}}'
      SLACK_MESSAGE: 'Workflow failed on ${{ github.repository }}'
