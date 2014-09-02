# Rails 快速上手
author
:   潘旻琦(孝达)

allotted-time
:   1h

theme
:   debian

# 日程安排

* Rails 核心技术
* 15 分钟创建博客演示
* 深入学习路线图


# Rails 核心技术

* 模型：Active Record
* 视图：Action View
* 控制器：Action Controller
* 路由：Action Dispatch

# Rails 核心技术

* 辅助：Active Support
* 邮件：Action Mailer
* 前端：sprockets
* 第三方扩展包

# 开始之前的提问时间
* 安装有没有问题？
* 创建项目有没有问题？
* 创建的目录结构看过了吗？

# Active Record

* 对象关系映射
* 数据库迁移
* 数据校验
* 对象生命周期管理
* 对象关联管理
* 数据查询

# 对象关系映射

| 类名  | 表名  |
|---|---|
| Post  | posts  |
| LineItem  | line_items  |
| Mouse  | mice  |
| STD  | 猜一猜 |
| B  | 猜一猜 |
| A::B  | 猜一猜 |

# 对象关系映射

| 类名  | 表名  |
|---|---|
| Post  | posts  |
| LineItem  | line_items  |
| Mouse  | mice  |
| STD  | stds |
| B  | bs |
| A::B  | bs |

# 特殊字段
* id、*_id
* created_at、updated_at
* lock_version
* type、*_type
* *_count
* 保留字（DangerousAttributeError）

# CRUD
* new、create、create!
* all、first、find_by、where
* update、update_all、update_attribute
* destroy、delete

# 数据库迁移

    class CreateProducts < ActiveRecord::Migration
      def change
        create_table :products do |t|
          t.string :name
          t.text :description
 
          t.timestamps
        end
      end
    end
{: lang="ruby"}

# 数据库迁移

    class ChangeProductsPrice < ActiveRecord::Migration
      def up
        change_table :products do |t|
          t.change :price, :string
        end
      end
 
      def down
        change_table :products do |t|
          t.change :price, :integer
        end
      end
    end
{: lang="ruby"}

# 数据库迁移

    $ bin/rails generate migration \
    AddPartNumberToProducts
    
    $ bin/rails generate migration \
    AddPartNumberToProducts part_number:string
    
    $ bin/rails generate migration \
    AddPartNumberToProducts part_number:string:index
    
    $ bin/rake db:rollback
    
    $ bin/rake db:rollback STEP=3
{: lang="bash"}


# 数据库迁移

    create_table(:apples) { |t| }
    add_column :products, :part_number, :string
    add_index :products, :part_number
    
    add_reference :products, :supplier,
          polymorphic: true, index: true
          
    create_join_table :products, :categories,
          table_name: :categorization
{: lang="ruby"}

# 数据校验

    validates :name, presence: true
    validates :password, confirmation: true
    validates :size, inclusion:
              { in: %w(small medium large) }
    validates :name, length: { minimum: 2 }
    validates :email, uniqueness: true
    validates :password, length: { in: 6..20 }
    validates :points, numericality: true
    validates :terms_of_service, acceptance: true
    validates_associated :books
{: lang="ruby"}

# 对象生命周期管理

    class User < ActiveRecord::Base
      validates :login, :email, presence: true

      before_validation :ensure_login_has_a_value

      protected
        def ensure_login_has_a_value
          if login.nil?
            self.login = email unless email.blank?
          end
        end
    end
{: lang="ruby"}

# 对象生命周期管理 (C)

* before_validation
* after_validation
* before_save
* around_save
* before_create
* around_create
* after_create
* after_save

# 对象生命周期管理 (R)

* after_initialize
* after_find

# 对象生命周期管理 (U)

* before_validation
* after_validation
* before_save
* around_save
* before_update
* around_update
* after_update
* after_save

# 对象生命周期管理 (D)

* before_destroy
* around_destroy
* after_destroy

# 对象关联管理

* belongs_to
* has_one
* has_many
* has_many :through
* has_one :through
* has_and_belongs_to_many

# 数据查询

    client = Client.find(10)
    # SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
    
    client = Client.take(2)
    # SELECT * FROM clients LIMIT 2
    
    client = Client.first
    # SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
    
    client = Client.last
    # SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1
{: lang="ruby"}

# 数据查询

    Post.none
    Client.find_by first_name: 'Lifo'
    Client.select(:name).distinct
    Client.order(created_at: :asc)
    Client.limit(5).offset(30)
    
    Order.select(
     "date(created_at) as ordered_date,
      sum(price) as total_price").
    group("date(created_at)").
    having("sum(price) > ?", 100)
{: lang="ruby"}

# 数据查询

    Client.find_by first_name: 'Lifo'
    Client.select(:name).distinct
    Client.limit(5)
    
    Order.select(
     "date(created_at) as ordered_date,
      sum(price) as total_price").
    group("date(created_at)").
    having("sum(price) > ?", 100)
{: lang="ruby"}

# Action View

    * ERB / jbuilder
    * 主模板 / 部分模板
    * content_for / yield
    * Remote

# ERB

    <h1>Names of all the people</h1>
    <% @people.each do |person| %>
      Name: <%= person.name %><br>
    <% end %>
{: lang="html"}

# jbuilder

    json.content format_content(@message.content)
    json.(@message, :created_at, :updated_at)

    json.comments @message.comments, :content, :created_at

    json.attachments @message.attachments do |attachment|
      json.filename attachment.filename
      json.url url_for(attachment)
    end
{: lang="ruby"}


# 主模板 / 部分模板

    <%= render "menu" %>
    
    <%= render partial: 'post',
        layout: 'box', locals: {post: @post} %>
    
    <% @products.each do |product| %>
      <%= render partial: "product",
          locals: {product: product} %>
    <% end %>
{: lang="html"}


# content_for / yield

    <html>
      <head>
        <title>Welcome!</title>
        <%= yield :special_script %>
      </head>
      <body>
        <p>Welcome! The date and time is <%= Time.now %></p>
      </body>
    </html>
{: lang="html"}

# content_for / yield

    <p>This is a special page.</p>
 
    <% content_for :special_script do %>
      <script>alert('Hello!')</script>
    <% end %>
{: lang="html"}

# Remote

    <%= link_to "a post", @post, remote: true %>
{: lang="html"}

# Remote

    $(function() {
      return $("a[data-remote]").on(
       "ajax:success", function(e) {
        return alert("The post was deleted.");
      });
    });
{: lang="js"}


# Remote

    <%= form_for(@post, remote: true) do |f| %>
      ...
    <% end %>
{: lang="html"}

# Remote

    $(document).ready(function() {
      return $("#new_post").on("ajax:success",
      function(e, data, status, xhr) {
        return $("#new_post").append(xhr.responseText);
      }).on("ajax:error",
       function(e, xhr, status, error) {
        return $("#new_post").append("<p>ERROR</p>");
      });
    });
{: lang="js"}
