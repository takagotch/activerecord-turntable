### activerecord-turntable
---

https://github.com/drecom/activerecord-turntable

```sh
gem 'activerecord-turntable', '~> 4.3.0'
bundle install
bundle exec rails g active_record:turntable:install


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


```
```yml
#turntable.yml
development:

#database.yml
connection_spec: &spec

# config/turntable.yml
deevelopment:

# config/database.yml
...
  shards:
  
# mysq1
development:

development:

```

