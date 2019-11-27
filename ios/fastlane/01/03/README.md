[TOC]



## 1. ==Appfile== 默认 ==全局== 配置

App/fastlane/Appfile:

```ruby
app_identifier "net.sunapps.1" # The bundle identifier of your app
apple_id "felix@krausefx.com"  # Your Apple email address

# You can uncomment the lines below and add your own
# team selection in case you're in multiple teams
# team_name "Felix Krause"
# team_id "Q2CBPJ58CA"

# To select a team for App Store Connect use
# itc_team_name "Company Name"
# itc_team_id "18742801"
```



## 2.  platform 单独配置

```ruby
for_platform :mac do
  app_identifier "com.forplatform.mac"

  for_lane :release do
    app_identifier "com.forplatform.mac.forlane.release"
  end
end
```



## 3. lane 单独配置

```ruby
for_lane :enterprise do
  app_identifier "com.forlane.enterprise"
end
```



## 4. language

```ruby
# 全局
locales ['en-US', 'fr-FR', 'ja-JP']

# 给某个 lane
for_lane :screenshots_english_only do
  locales ['en-US']
end

# 给某个 lane
for_lane :screenshots_french_only do
  locales ['fr-FR']
end
```