# Siteshot

> Скрипт для создания снапшотов сайта в формате html страничек.
> Используется для SEO в одностраничниках построенных на таких фреймворках как Angluar.js, Backbone.js и другие


`Siteshot` использует xml файл со списком страничек для которых нужно сделать снапшоты, поэтому первым делом на нужно добавить и настроить генератор карты сайта, для этого мы будем использовать гем `sitemap_generator`
И еще нам потребуется `whenever` для добавления задач в крон а в данном случае для ротации xml файла.

### Установка `sitemap_generator`

  1. Добавляем в `Gemfile`

    ```ruby
    gem 'sitemap_generator'
    gem 'whenever', require: false
    ```

  2. Добавляем в `config/application.yml`

    ```
    host: 'YOUR_DOMAIN'
    ```

  3. Добавляем конфигурацию `config/sitemap.rb` и настраиваем под нужны своего проекта

    ```ruby
    # Set the host name for URL creation
    SitemapGenerator::Sitemap.default_host = ENV['host'].present? ? ENV['host'] : 'YOUR_DOMAIN'
    SitemapGenerator::Sitemap.sitemaps_path = 'system'

    SitemapGenerator::Sitemap.create do
      add 'product'
      add 'horoscope'
      add 'konkurs'
      add 'prizes'
      add 'articles/ideas', changefreq: 'weekly'
      add 'articles/evening', changefreq: 'weekly'
      add 'articles/wedding', changefreq: 'weekly'

      Article.find_each do |article|
        add "article/#{article.slug}", lastmod: article.updated_at
      end
    end
    ```

  4. Добавляем в `Capfile`

    ```ruby
    # Cron jobs
    require 'whenever/capistrano'

    # Sitemap
    require 'capistrano/sitemap_generator'
    ```

  5. Создаем расписание `config/schedule.rb`

    ```ruby
    every 1.day do
      rake 'sitemap:refresh'
    end
    ```

  После этих действий у нас появляется в списке команд `cap -T` команды

    ```
    cap deploy:sitemap:clean           # Clean up sitemaps in sitemap_generator path
    cap deploy:sitemap:create          # Create sitemap without pinging search engines
    cap deploy:sitemap:refresh         # Create sitemap and ping search engines
    ```


  5. Деплоимся и запускаем генерацию xml

    ```
    cap YOUR_ENV deploy && cap YOUR_ENV deploy:sitemap:create
    ```

    После этого мы видим в `public/system/sitemap.xml.gz`

  * Поздравляю вы настроили `sitemap_generator` а так же ротацию

### Siteshot

  1. Установка

    ```
    sudo apt-get install npm nodejs-legacy
    curl https://raw.githubusercontent.com/creationix/nvm/v0.26.0/install.sh | bash
    source ~/.bashrc
    nvm install 0.10.36
    nvm alias stable 0.10.36
    npm install phantomjs@1.8.1-3 -g
    git clone https://github.com/donbobka/siteshot/
    cd siteshot
    git checkout feature/add_page_modifier
    npm pack
    npm install siteshot-0.0.3.tgz -g
    ```

  2. Заходим на сервер деплоя создаем в корне проекта папку `shadow-copy`

    ```
    mkdir /var/www/APPLICATION_NAME/shadow-copy
    ```

  3. В `shadow-copy` создаем конфигурационные файлы

    `config.js`

    ```js
    module.exports = {
      snapshotDir: "/var/www/APPLICATION_NAME/shadow-copy/snapshots",
      sitemap: "/var/www/APPLICATION_NAME/shadow-copy/sitemap.xml",
      delay: 5000,
      pageModifier: function(page, callback) {  page.evaluate(function() { $('meta[name=fragment]').remove() }); callback(); }
    }
    ```

    `make-copy.rb`

    ```ruby
    #!/usr/bin/env ruby

    `wget #{ENV['SITEMAP_URL']}`
    `gzip -df ./sitemap.xml.gz`
    `siteshot`
    ```

    Назначаем права для `make-copy.rb`

    ```
    chmod 0775 make-copy.rb
    ```

    `config.js` хранятся настройки для siteshot'a в данном примере указаны:
      snapshotDir - папка для хранения снапшотов
      sitemap - путь до карты сайта
      delay - задержка при создании снапшота для того чтобы страница успела загрузиться и javascript успел отработать
      pageModifier - удаляется тэг meta='fragment', он не нужен для СЕО

    `make-copy.rb` скачивает нашу карту сайта, затем разархивирует ее и запускает siteshot для того чтобы сделать снапшоты нашего сайта

  4. Добавляем в cron

    ```
    0 4 * * * /bin/bash -l -c 'cd /var/www/APPLICATION_NAME/shadow-copy && SITEMAP_URL=YOUR_DOMAIN/system/sitemap.xml.gz ./make-copy.rb >./siteshot.log'
    ```


  5. Добавляем в nginx

    После location ~* ^/(api|assets|system|admin|rich)

    ```
    location ^~ /seo {
      root /var/www/APPLICATION_NAME/shadow-copy/snapshots;
      rewrite ^/seo(.*)$ $1 break;
      try_files $uri $uri.html $uri/index.html =404;
    }
    ```

    После location @ss или @backend

    ```
    location @index {
      set $prerender 0;
      if ($http_user_agent ~* "baiduspider|twitterbot|facebookexternalhit|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator") {
        set $prerender 1;
      }
      if ($args ~ "_escaped_fragment_") {
        set $prerender 1;
      }
      if ($http_user_agent ~ "PhantomJS") {
        set $prerender 0;
      }

      if ($prerender = 1) {
        rewrite ^ /seo$uri last;
      }
      if ($prerender = 0) {
        root /var/www/APPLICATION_NAME/cs/current;
        rewrite ^ /index.html break;
      }
    }
    ```

    ```
    sudo service nginx restart
    ```

 # THAT'S ALL !!!
