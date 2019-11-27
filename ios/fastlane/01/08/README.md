```ruby
cd toolbox
bundle exec fastlane pipeline_approve

# 捕获 lane 异常退出
if [ $? -ne 0 ]; then
	echo "pipeline_approve exec error" 1>&2
	exit 1
fi

cd ..
```

