# Introduction #

You can of course use ActiveMessaging as a regular Rails plugin, as explained in the TenMinuteIntroduction.  Alternatively, you can use it as a gem.  Using ActiveMessaing as a gem in Rails has a few advantages:

  * Versioned releases.  The plugin installation takes a snapshot of the current source repository possibly in mid-change.  The gem releases are more stable.
  * Dependency enforcement.  Each ActiveMessaging gem defines what gems it relies on and what versions of these gems are required.  This is also enabled during updates.
  * Share space.  Multiple Rails applications can use ActiveMessaging from the same gem installation.  You can still have a copy of all the required gems in your projects by running 'rake freeze:gems', if you want.


# Howto #

**Define ActiveMessaging as a required gem in your config/environment.rb file:
```
Rails::Initializer.run do |config|
  ...
  config.gem 'activemessaging'
  ...
end
```**

**Add gems for the desired brokers:
```
Rails::Initializer.run do |config|
  ...
  config.gem 'stomp'
  ...
end
```**

**Install the required gems if they aren't installed already
```
rake gems:install
```**

