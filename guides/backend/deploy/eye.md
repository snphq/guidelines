# Eye

> Eye является инструментом мониторинга процессов. Вдохновленный от BluePill и Богом.


## Настройка

1. Первым делом добавляем в `Gemfile`

  ```ruby
  gem 'eye'
  gem 'capistrano-eye'
  ```
  
2. Добавляем в `Capfile`

  ```ruby
  require 'capistrano/eye'
  ```

3. Добавляем в `config/deploy.rb`

  ```ruby
  set :default_env, { path: "~/.rbenv/shims:~/.rbenv/bin:$PATH" }
  ``` 

4. Создаем в `config` папку eye затем добавляем и настроиваем конфигурационные файлы

  `config/eye/common.rb`

  ```ruby
  ROOT = '/var/www/your_app_name/ss'
  CURRENT = File.expand_path(File.join(ROOT, %w{current}))
  SHARED = File.expand_path(File.join(ROOT, %w{shared}))
  LOGS = File.expand_path(File.join(ROOT, %w{shared log}))
  PIDS = File.expand_path(File.join(ROOT, %w{shared tmp pids}))

  Eye.config do
    logger "#{LOGS}/eye.log"
    logger_level Logger::ERROR
  end

  Eye.application 'application_name' do
    env 'RAILS_ENV' => RAILS_ENV
    working_dir CURRENT
    triggers :flapping, :times => 10, :within => 1.minute

    process :puma do
      daemonize true
      pid_file "#{PIDS}/puma.pid"
      stdall "#{LOGS}/#{RAILS_ENV}.log"

      start_command "#{ENV['HOME']}/.rbenv/bin/rbenv exec bundle exec puma -C #{SHARED}/puma.rb --daemon"
      stop_command "kill -TERM {{PID}}"
      restart_command "kill -USR2 {{PID}}"

      start_timeout 15.seconds
      stop_grace 10.seconds
      restart_grace 10.seconds

      checks :cpu, :every => 30, :below => 80, :times => 3
      checks :memory, :every => 30, :below => 200.megabytes, :times => [3,5]
    end

    process :sidekiq do
      start_command "#{ENV['HOME']}/.rbenv/bin/rbenv exec bundle exec sidekiq --index 0 --environment #{RAILS_ENV}"
      pid_file 'tmp/pids/sidekiq.pid'
      stdall 'log/sidekiq.log'
      daemonize true
      stop_signals [:USR1, 0, :TERM, 10.seconds, :KILL]

      check :cpu, every: 30, below: 100, times: 5
      check :memory, every: 30, below: 300.megabytes, times: 5
    end
  end  
  ```

  `config/eye/testing.eye`

  ```ruby
  RAILS_ENV = 'testing'
  Eye.load('common.rb')
  ```
  
  `config/eye/production.eye`

  ```ruby
  RAILS_ENV = 'production'
  Eye.load('common.rb')
  ```

5. В `config/deploy/production.rb` добавляем

  ```ruby
  set :eye_config, -> { 'config/eye/production.eye' }
  ```

6. В `config/deploy/testing.rb` добавляем

  ```ruby
  set :eye_config, -> { 'config/eye/testing.eye' }
  ```  

7. Из `Capfile` убираем `capistrano/puma` и `capistrano/sidekiq`

8. Добавьте в cron

  ```
  @reboot /usr/bin/env eye load /var/www/your_app_name/ss/current/config/eye/production.eye
  ```

## DEPLOY & ENJOY !!!
