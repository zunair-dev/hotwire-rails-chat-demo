# Step by Step tutorial

## Create a new rails application with hotwire-rails lib

1. `rails new chat --skip-javascript`
2. add `gem hotwire-rails` to Gemfile
3. `bundle`
4. `rails hotwire:install`

## Build the application without using hotwire features first

1. `rails g scaffold room name:string`
2. `rails g model message room:references content:text`
3. `rails db:migrate`
4. Update [routes.rb](config/routes.rb)
5. Add `has_many :messages` to room.rb
6. Create a MessagesController with:
```ruby
class MessagesController < ApplicationController
  before_action :set_room, only: %i[ new create ]

  def new
    @message = @room.messages.new
  end

  def create
    @message = @room.messages.create!(message_params)

    respond_to do |format|
      format.html { redirect_to @room }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_room
      @room = Room.find(params[:room_id])
    end

    # Only allow a list of trusted parameters through.
    def message_params
      params.require(:message).permit(:content)
    end
end
```
7. Create the messages new.html.erb template with:
```
<h1>New Messages</h1>

<%= form_with(model: [ @message.room, @message ]) do |form| %>
    <div class="field">
        <%= form.text_field :content %>
        <%= form.submit "Send" %>
    </div>
<% end %>

<%= link_to 'Back', @message.room %>
```
8. Create the partial template _message.html.erb with:
```
<p id="<%= dom_id message %>">
    <%= message.created_at.to_s(:short) %>: <%= message.content %>
</p>
```
9. Edit room's show.html.erb to present messages and link to new message:
```
<p id="notice"><%= notice %></p>

<p>
    <strong>Name:</strong>
    <%= @room.name %>
</p>

<p>
    <%= link_to 'Edit', edit_room_path(@room) %> |
    <%= link_to 'Back', rooms_path, "data-turbo-frame": "_top" %>
</p>

<div id="messages">
  <%= render @room.messages %>
</div>

<%= link_to 'New Message', new_room_message_path(@room) %>
```
10. Start the server and try it out

## Adding Turbo drive to the app

1. Update room's show.html.erb to use turbo_frame_tag
```
<p id="notice"><%= notice %></p>

<%= turbo_frame_tag "room" do %>
    <p>
        <strong>Name:</strong>
        <%= @room.name %>
    </p>

    <p>
        <%= link_to 'Edit', edit_room_path(@room) %> |
        <%= link_to 'Back', rooms_path, "data-turbo-frame": "_top" %>
    </p>
<% end %>

<div id="messages">
  <%= render @room.messages %>
</div>

<%= turbo_frame_tag 'new_message', src: new_room_message_path(@room), target: '_top' %>
```
2. Also update the room's [edit.html.erb](app/views/rooms/edit.html.erb)
3. And message's new.html.erb with:
```
<h1>New Messages</h1>

<%= turbo_frame_tag 'new_message', target: '_top' do %>
    <%= form_with(model: [ @message.room, @message ]) do |form| %>
        <div class="field">
            <%= form.text_field :content %>
            <%= form.submit "Send" %>
        </div>
    <% end %>
<% end %>

<%= link_to 'Back', @message.room %>
```
4. Add `format.turbo_stream` to the respond_to block of the MessagesControler#create
5. Create a new file create.turbo_stream.erb with:
```
<%= turbo_stream.append "messages", @message %>
```

## Add Stimulus to the app

1. Create a [reset_form_controller.js](app/assets/javascripts/controllers/reset_form_controller.js)
2. Update [messages' new.html.erb](app/views/messages/new.html.erb) to use the new reset javascript method
3. Add WebSocket stream to have messages automatically presented on screen whenever they're created/updated/removed
4. Add turbo_stream_from tag to room's [show.html.erb](app/views/rooms/show.html.erb)
5. Note we're also extracting a [room partial](app/views/rooms/_room.html.erb)
6. Add broadcasts_to to both [message.rb](app/models/message.rb) and [room.rb](app/models/room.rb)
7. Now that's using broadcasts the create.turbo_stream.erb file isn't used anymore, just erase it.
