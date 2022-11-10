# Using Action Cable with React and Rails.
There are no tutorials for this, so I am making one.

This Guide assumes you:
- already have a functioning app with React on top of Rails.
- have a User table that:
  - implements `has_secure_password` from
  - stores logged in user_id inside `session[:user_id]`

If you dont have any of that, consider taking a look at my [react rails template](https://github.com/CodeF53/react_rails_template)

For a finished project that implements this tutorial, look at my [react rails chat app](https://github.com/codeF53/react_rails_chat), which is built off of that template.

## Postgresql
### installing postgresql
TODO

### switching to it
in your `/Gemfile` switch out sqlite3 for postgres:
```diff
- # sqlite3 as database for Active Record
- gem 'sqlite3', '~> 1.4'
+ # Use postgresql as the database for Active Record
+ gem 'pg', '~> 1.1'
```

replace your `/config/database.yml` with a postgres one
```yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  database: APPNAME_development

test:
  <<: *default
  database: APPNAME_test

production:
  <<: *default
  database: APPNAME_production
  username: APPNAME
  password: <% ENV["APPNAME_DATABASE_PASSWORD"] %>
```

## Adding action cable
### Redis
Install redis:
```
sudo apt install redis-server
```

Add redis to your `/Gemfile`
```ruby
# Use Redis adapter to run Action Cable in production
gem 'redis', '~> 4.0'
```

From now on when starting your server you have to start the redis server
```
redis-server
rails s
npm start
```

### Action cable
Inside `/config/application.rb` uncomment the require for action cable. (this is typically on line 14)
```ruby
require "action_cable/engine"
```

Add an action cable route to your `/config/routes.rb`
```ruby
Rails.application.routes.draw do
  # ActionCable Magic
  mount ActionCable.server => '/cable'
  ...
  # The rest of your routes
end
```

Create `/config/cable.yml`:
```yml
development:
  adapter: redis

test:
  adapter: test

production:
  adapter: redis
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: phase_4_project_guidelines_production
```

Create `/app/channels/application_cable/channel.rb`
```rb
module ApplicationCable
  class Channel < ActionCable::Channel::Base
  end
end
```

Create `/app/channels/application_cable/connection.rb`
```rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private

    def find_verified_user
      user = User.find(cookies.encrypted['_session_id']['user_id'])

      return user unless user.nil?

      reject_unauthorized_connection
    end
  end
end
```

## Adding a channel and some broadcasts

### Channel
This is channel that your frontend will end up connecting to. They handle subscribing to and unsubscribing from the handshake data stream thingy.

Create `/app/channels/things_channel.rb` where `thing` is one of your models.

You will later use the `thing` model in a broadcast function, so make sure you have access to it.

`things_channel.rb`
```rb
class ThingsChannel < ApplicationCable::Channel
  def subscribed
    stop_all_streams
    thing = Thing.find(params[:thing_id])
    stream_for thing
  end

  # You can add a received function here,
  # but I dont know what it does

  def unsubscribed
    stop_all_streams
  end
end
```

In my case, I am going to make `rooms_channel.rb`

`rooms_channel.rb`, as an example
```rb
class RoomsChannel < ApplicationCable::Channel
  def subscribed
    stop_all_streams
    room = Room.find(params[:room_id])
    stream_for room
  end

  # You can add a received function here,
  # but I dont know what it does, I didn't need it.

  def unsubscribed
    stop_all_streams
  end
end
```

### Broadcasting data to the channel
Broadcasts take a Model and a hash, then send it to all things subscribed to that Model's channel. You can put broadcasts basically anywhere in your code.

```rb
ThingsChannel.broadcast_to(
  thing, # an instance of your model, ex Thing.find(id)
  hash # any data
)
```

In my case, I want to broadcast data to the relevant `Room`'s channel when a new message created or deleted in it.

`/app/controllers/messages_controller.rb`:
```rb
# POST /messages
def create
  room = Room.find(params[:room_id])

  message = Message.new(text_content: params[:text_content])
  message.user = @current_user
  message.room = room
  message.save!

  # after successfully creating the message, update the room that it was in
  broadcast room

  render json: message, status: :created
end

# DELETE /messages/1
def destroy
  message = Message.find(params[:id])
  room = message.room
  message.destroy

  # after successfully yeeting the message, update the room that it was in
  broadcast room
end

private

def broadcast(room)
  # ActiveModelSerializers::SerializableResource.new(object).as_json
  # returns the same thing sent by render json: object
  RoomsChannel.broadcast_to(room, ActiveModelSerializers::SerializableResource.new(room).as_json)
end
```

This is a pretty lazy implementation, as I just always re-serialize the entire room object and send just that.

This has the benefit of needing less thought on how you broadcast data in the backend and  on how you process the data you get on the frontend.

A more smart implementation would be something like this
```rb
# POST /messages
def create
  ...
  # after successfully creating the message, tell the room to add it
  RoomsChannel.broadcast_to(room, { new_message: message })
  ...
end

# DELETE /messages/1
def destroy
  ...
  # after successfully yeeting the message, tell the room to remove it
  RoomsChannel.broadcast_to(room, { remove_message: params[:id] })
  ...
end
```
This would have the benefit of being more resource friendly on both the frontend and backend. Its just a bit more work to implement.

I am going to be giving examples for the lazy broadcast method from now on.

## React side:
### Installing Action Cable Provider
Install [JackHowa's fork of react-actioncable-provider](https://github.com/JackHowa/react-actioncable-provider), unless you are reading this sometime in the future like 2024, in that case [take a look at the network graph to make sure you are using the most well maintained branch](https://github.com/JackHowa/react-actioncable-provider/network):
```
cd client
npm install --save @thrash-industries/react-actioncable-provider
```

### Getting the Cable:
Inside your `/client/src/index.js` add some stuff to initialize a `cableApp` and pass `cable` down into your `App`.
```js
import React from 'react';
...
import ActionCable from "actioncable";

const cableApp={}
cableApp.cable=ActionCable.createConsumer("/cable")

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App cable={cableApp.cable} />
  </React.StrictMode>
);
```

### Passing the cable down
Pass the Cable down through your app similar to how you would do with state, repeating down to the component you need the stuff in:
```js
...
import Room from "./pages/Room";

export default function App({ cable }) {
  ...
  return <div id="app" className="col centered">
    ...
    <Route path="/room/:room_id" element={<Room cable={cable}/>}/>
    ...
  </div>
}
```

### Using the cable


```js
...
export default function Room({ cable }) {
  const { room_id } = useParams()

  // Add some state for your the latest data from the Channel
  // optionally with defaults so your program doesn't die before its gotten data
  const [roomObj, setRoomObj] = useState({messages:[]})

  useEffect(() => {
    // manually fetch to get the initial state
    fetch(`/rooms/${room_id}`).then(r=>r.json().then(d=>setRoomObj(d)))

    // subscribe to updates for room
    cable.subscriptions.create({ channel: "RoomsChannel", room_id: room_id },
    {
      // optionally do some stuff with connects and disconnects
      connected: () => console.log("room connected!"),
      disconnected: () => console.log("room disconnected!"),
      // update your state whenever new data is received
      received: (updatedRoom) => setRoomObj(updatedRoom)
    }
  )}, [cable, room_id])

  return <div id="room" className="col">
    {/* Use your live data somehow */}
  </div>
}
```