# Инструкция, как работать с гемом Carrierwave (загрузка изображений в Ruby on Rails)

- [GitHub - carrierwaveuploader/carrierwave](https://github.com/carrierwaveuploader/carrierwave)
- [GitHub - minimagick/minimagick](https://github.com/minimagick/minimagick)

**Добавить в Gemfile:**

```ruby
gem 'carrierwave', '~> 1.0'
gem "mini_magick"
```

**Генерация:**

```bash
rails generate uploader Picture
```

**Настройка конфигурации app/uploaders/picture_uploader.rb:**

- Раскомментировать строку:

```ruby
include CarrierWave::MiniMagick
```

- Выбрать путь к файлам в store_dir

- Можно настроить несколько версий для изображения в строке: version

```ruby
version :large do
  process resize_to_fit: [1024, 1024]
end

version :title do
  process resize_to_fit: [600, 450]
end

version :thumb do
  process resize_to_fit: [300, 300]
end
```

- Валидация формата файлов:

```ruby
def extension_whitelist
  %w(jpg jpeg gif png)
end
```

**Проведём миграцию, прикрепим изображение к нашим страницам**

Предположим, что у нас есть сущность Project - [инструкция, как создать сущность Project](https://github.com/krdprog/rails_help_for_novice/blob/master/README.md)

```bash
rails g migration add_picture_to_projects picture:string
```
```bash
rake db:migrate
```

**Пропишем в модели app/models/project.rb**

```ruby
class Project < ApplicationRecord
  mount_uploader :picture, PictureUploader
end
```

**Добавим в форму (в /app/views/projects/new.html.erb):**

```html
<p>
  <label for="project_picture">Picture:</label><br>
  <%= f.file_field :picture %>
</p>
```
и там же добавить в строку:
```ruby
<%= form_for :project, html: { multipart: true },
```

И, в контроллере /app/controllers/projects_controller.rb укажем разрешённый параметр :picture

```ruby
def project_params
  params.require(:project).permit(:title, :body, :picture)
end
```

**Ещё нам надо вывести картинку в show.html.erb**

```ruby
<p><%= image_tag @project.picture.url(:title), class: 'img-show', alt: @project.title if @project.picture? %></p>

<p><%= image_tag @project.picture.url(:thumb), class: 'img-show', alt: @project.title if @project.picture? %></p>
```

**Можно добавить CSS:**

```css
.img-show {
  display: block;
  margin-bottom: 2rem;
}
```

Запустим `rails s` и откроем в браузере `http://localhost:3000/projects/new`, в форме появится кнопка загрузки фотографий.

