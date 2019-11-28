[TOC]



# Load own actions from external folder

Add this to the top of your **Fastfile**.

```ruby
actions_path '../custom_actions_folder/'

lane :hello do |options|
	# ....
end
```

