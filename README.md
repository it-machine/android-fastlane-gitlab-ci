# Android Fastlane for Gitlab-CI

В данный образ включены Ruby, Android SDK и все необходимые пакеты для сборки Android приложения в Gilab CI. 

## Текущие версии:
1. Ruby - 2.3.0
2. Sdk tools - 25.2.3
3. Build tools - 25.0.1
4. Target sdk - 25

## Пример настроек .gitlab-ci.yml

```
image: "strollager/android-fastlane-gitlab-ci"

cache:
  paths:
    - .gradle/
    
before_script:
  - eval "$(rbenv init -)"
  
stages:
  - build
  
build:
  stage: build
  only:
    - develop
  script:
    - cd /builds/YOUR_GROUPNAME/YOUR_PROJECT_NAME
    - fastlane beta api_token:$api_token build_secret:$build_secret develop_group:$develop_group
```

## Пример настроек Fastlane

```
fastlane_version "1.111.0"

default_platform :android

platform :android do
  before_all do
    # ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
  end

  desc "Runs all the tests"
  lane :test do
    gradle(task: "test")
  end

  desc "Submit a new Beta Build to Crashlytics Beta"
  lane :beta do |options|
    gradle(
      task: 'assemble',
      build_type: 'Debug'
    )

    crashlytics(
      groups: options[:develop_group],
      api_token: options[:api_token],
      build_secret: options[:build_secret]
    )

    slack(
      # ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
    )
    
  end

  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "assembleRelease")
    supply
  end

  # You can define as many lanes as you want

  after_all do |lane|
    # This block is called, only if the executed lane was successful

    # slack(
    #   message: "Successfully deployed new App Update."
    # )
  end

  error do |lane, exception|
    # slack(
    #   message: exception.message,
    #   success: false
    # )
  end
end
```
