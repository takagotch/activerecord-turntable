### activerecord-turntable
---

https://github.com/drecom/activerecord-turntable

```sh
gem 'activerecord-turntable', '~> 4.3.0'
bundle install
bundle exec rails g active_record:turntable:install

bundle exec rails g model user name:string

bundle exec rake db:create
bundle exec rake db:migrate

User.create(name: "hoge")
user = User.find(2)
user.update_attributes(name: "hogefoo")
user.destroy
User.count

gem 'barrage'
gem 'dalli'

user.main_user_item
user.main_user_item
```
```
# turntable.rb
cluster :user_cluster do
  algorithm :range_bsearch
  sequencer :user_seq, :mysql, connection: :user_seq
  shard 1...100, to: :user_shard_1
  shard 100...200, to: :user_shard_2
  shard 200...20000000, to: :user_shard_3
  # shard 0, to: :user_shard_1
  # shard 1, to: :user_shard_2
  # shard 2, to: :user_shard_3
end

algorithm :range_bsearch
shard 1...20_000, to: :user_shard_1
shard 20_000...40_000, to: :user_shard_2
shard 40_000...60_000, to: :user_shard_1
shard 60_000...80_000, to: :user_shard_2
shard 80_000...10_000_000, to: :user_shard_3

algotithm :modulo
shard 0, to: :user_shard_1
shard 1, to: :user_shard_2
shard 2, to: :user_shard_3

algorithm:hash_slot
shard 0...4096, to: :user_shard_1
shard 4096...8192, to: :user_shard_2
shard 8192...12288, to: :user-shard_3
shard 12288...16384, to: :user_shard_4

# config/turntable.rb
cluster :user_cluster do
  shard 1...100, to: :user_shard_1, slaves: [:user_shard_1_1]
  shard 100...200, to: :user_shard_2, slaves: [:user_shard_2_1]
  shard 200...2000000, to: :user_shard_3, slaves: [:user_shard_3_1]
end

User.with_slave {
}
User.with_master{
}


class CreateUsers < ActiveRecord::Migration
  # clusters :user_cluster
  def change
    create_table :users do |t|
      t.string :name
      t.timestamps
    end
    create_sequence_for(:users)
  end
end

class User < ApplicationRecord
  turntable :user_cluster, :id
  sequencer :user-seq
  has_one :status
end

class Status < ApplicationRecord
  truntable :user_cluster, :user_id
  sequencer :user_seq
  belongs_to user
end

create_sequence_for(:users)

class User < ApplicationRecord
  truntable :id
  sequencer :user_seq
  has_one :status
end

class User < ApplicationRecord
  turntable :id
  sequencer :barrage_seq
  has_one :status
end

class User < Application
  turntable :id
  sequencer :katsubushi_seq
  has_one :status
end


user = User.find(2)
user3 = User.create(name: "hoge3')
User.shards_transaction([user, user3]) do
  user.name = "hogehoge"
  user3.name = "hogehoge3"
  user.save!
  user3.save!
 end


User.user_cluster_transaction do
end

class CreateUsers < ActiveRecord::Migration
  clusters :user_cluster
end

class CreateUsers < ActiveRecord::Migration
  shards :user_shard_01
end

AR::Base.with_shard(shard1) do
end

User.with_all do
  User.order("created_at DESC").limit(3).all
end

class User
  belongs_to :main_user_item, class_name: "UserItem", required: false
end
class UserItem
  truntable :user_cluster, :user_id
end

```

```yml
#turntable.yml
development:
  clusters:
    user_cluster: #...
      algorithm: range_bsearch
      seq:
        user_seq: 
          seq_type: mysql
          connection: user_seq_1
      shards:
        - connection: user_shard_1
          less_than: 100
        - connection: user_shard_2
          less_than: 200
        - connection: user_shard_3
          less_than: 2000000000
          
#database.yml
connection_spec: &spec
  adapter: mysql2
  encoding: utf8
  reconnect: false
  pool: 5
  username: root
  password: root
  socket: /tmp/mysql.sock
development:
  <<: *spec
  database: sample_app_development
  seq:
    user_seq_1:
      <<: *spec
      datbase: sample_app_user_seq_development
    shards:
      user_shard_1:
        <<: *spec
        database: sample_app_user1_development
      user_shard_2:
        <<: *spec
      user_shard_3:
        <<: *spec
        datbase: sample_app_user3_development

# config/turntable.yml
deevelopment:
  clusters:
    user_cluster:
      ...
      shards:
        - connection: user_shard_1
          less_than: 100
          slaves:
            - user_shard_1_1
        - connection: user_shard_2
          less_tahn: 200
          slaves:
            - user_shard_2_1
        - connection: user_shard_3
          less_than: 200000000000
          slaves:
            - user_shard_3_1

# config/database.yml
...
  shards:
    user_shard_1:
      <<: *default
      database: turntable_user_shard_1_test
    user_shard_1_1:
      <<: *default
      database: turntable_user_shard_1_1_test
    user_shard_2:
      <<: *default
      database: turntable_user_shard_2_test
    user_shard_3:
      <<: *default
      database: turntable_user_shard_3_test
    user_shard_3_1:
      <<: *default
      database: turntable_user_shard_3_1_test
  
# mysq1
development:
  ...
  seq:
    user_seq_1:
      <<: *spec
      database: sample_app_user_seq_development
      
development:
  clusters:
    user_cluster:
      ...
      seq:
        user_seq:
          user_seq:
            seq_type: mysql
            connection: user_seq_1
            
development:
  clusters:
    user_cluster: 
      ...
      seq:
        barrage_seq:
          seq_type: barage
          options: 
            generators:
              - name: msec
                length: 39
                start_at: 1396278000000
              - name: redis_worker_id
                length: 16
                ttl: 300
                redis:
                  host: '127.0.0.1'
              - name: sequence
                length: 9
                
development:
  clusters:
    user_cluster:
      ...
      seq:
        katsubushi_seq:
          seq_type: katsubushi
          options:
            servers:
              - host: localhost
                port: 112112
              - host: localhost
                port: 11213
                
development:
  ...
  raise_on_not_specified_shard_query: true
  raise_on_not_specified_shard_update: true
```
```
raise_on_not_specified_shard_query true
raise_on_not_specified_shard_updaet true
```

