Doctor `has_many` Patients `:through` Appointments
=========================================

The has_many :through association is one of the most useful associations in Rails. It is a way of relating two models together *through* another model. For instance, a mentor may have many mentees and a mentee may have many mentors, all linked together through mentorships. The three models there would be mentors, mentees, and mentorships. 

Or doctors may have many patients and patients may have many doctors and they could all be linked together through their appointments. 

==============
First step, create our new project, called `doctor_app`.

        $ rails new doctor_app -d mysql

Next, we will create the three models we need for our `has_many :through` association. To make our lives easier, we'll use scaffolding in order to create the associated views and controllers at the same time.

        $ rails g scaffold doctor name:string
        $ rails g scaffold patient name:string
        $ rails g scaffold appointment reason:string doctor_id:integer patient_id:integer
        
With our new models, we need to run the migrations to create the necessary tables in our database, so:

        $ rails db:create db:migrate
        
And now let's make the associations. This is done in the model files. First up, `doctor.rb` (in app/models directory) should look like this:

        class Doctor < ActiveRecord::Base
          has_many :appointments
          has_many :patients, through: :appointments
        end
        
Notice how the syntax makes it very clear what this association is all about. Pretty cool! Then, the `patient.rb` model file:

        class Patient < ActiveRecord::Base
          has_many :appointments
          has_many :doctors, through: :appointments
        end
				
And, finally, set up the `appointments.rb` model file like this:

        class Appointment < ActiveRecord::Base
          belongs_to :doctor
          belongs_to :patient
        end

We're almost ready to take a look at the app. Just a couple of steps before we fire it up in our browser. We want the appointments index page to be the root of our app, so open up `config/routes.rb` and add this line somewhere (toward the top is nice):

        root 'appointments#index'
        
Then, start the local server in the `doctor_app` main directory:

        $ rails s
        
Open your browser, and go to the address `http://localhost:3000`. This is what you should see:

![](https://i.imgur.com/gRL7fs0.png)

Cool! Could it be that easy? Well, almost. If we click "New Appointment", we get this:

![](https://i.imgur.com/sgQ8N9X.png)

(It may look a little different, depending on your browser.) You'll notice that you're only allowed to enter numbers, though, which isn't exactly what we want.

![](https://i.imgur.com/S2LuKRX.png)

Those numbers are Doctor and Patient ID's, though none have been created yet. What we need are a bunch of doctor and patient names. Let's put some links in to create some doctors and patients. We need to head over to the views and do some work there. First up, `app/views/appointments/index.html.erb`. Add the following link code to the top of that file:

        <%= link_to 'New Doctor', new_doctor_path %>
        <%= link_to 'New Patient', new_patient_path %>
        
Take a look, reload http://localhost:3000 and we should see:

![](https://i.imgur.com/hkG4B26.png)

Ah ha, now we can start making some doctors and patients! First, let's make a bunch of doctors:

![](https://i.imgur.com/yGivKbM.png)

And then, create a slew of patients, all eager to become powerful doctors (go back to http://localhost:3000 to get to the "New patient" link):

![](https://i.imgur.com/5okFaTj.png)

Now we need to make it easy to pair the doctors and patients up through appointments, so let’s use their names rather than their ID’s through drop down menus. That code is found in the appointments `_form.html.erb` partial. Let’s change the end of that file from this:

      <div class="field">
    	  <%= f.label :doctor_id %><br />
    	  <%= f.number_field :doctor_id %>
      </div>
      <div class="field">
    	  <%= f.label :patient_id %><br />
    	  <%= f.number_field :patient_id %>
      </div>
      <div class="actions">
    	  <%= f.submit %>
      </div>
    <% end %>

to this:

          <div class="field">
            <%= form.select :doctor_id, Doctor.all.collect { |p| [ p.name, p.id ] } %>
          </div>
          <div class="field">
            <%= form.select :patient_id, Patient.all.collect { |p| [ p.name, p.id ] } %>
          </div>
          <div class="actions">
            <%= form.submit %>
          </div>
        <% end %>
        
Now we have drop down menus, populated with all of our doctors and patients, all ready to be synched up through their appointments!

![](https://i.imgur.com/j1H1vQN.png)

Create an appointment, and let’s see what happens…

![](https://i.imgur.com/j1H1vQN.png)

Hmm, we’re almost there, but we’re only seeing the doctor and patient ID’s in the appointment show view. Not very satisfying. Let’s change that so we can see their names.

In `views/appointments/show.html.erb`, change these lines:

        <p>
          <b>Doctor:</b>
          <%= @appointment.doctor_id %>
        </p>
        
        <p>
          <b>Patient:</b>
          <%= @appointment.patient_id %>
        </p>

to read like this:

        <p>
          <b>Doctor:</b>
          <%= @appointment.doctor.name %>
        </p>
        
        <p>
          <b>Patient:</b>
          <%= @appointment.patient.name %>
        </p>

And reload…

![](https://i.imgur.com/5OuIJar.png)

Much better! But, clicking “Back” to get back to the appointments index, we see those darn ID’s again.

![]()

To change that, let’s change the appropriate `views/appointments/index.html.erb` lines to read from this:

        <% @appointments.each do |appointment| %>
          <tr>
        	<td><%= appointment.doctor_id %></td>
        	<td><%= appointment.patient_id %></td>

to this:

        <% @appointments.each do |appointment| %>
          <tr>
        	<td><%= appointment.doctor.name %></td>
        	<td><%= appointment.patient.name %></td>

And reloading the appointments index, we see:

![](https://i.imgur.com/QTNb5IJ.png)

That’s what we want to see! Make a few more associations, and we can see all the appointments we need:

![](https://i.imgur.com/QTNb5IJ.png)

<br>

## YOU DO

Using this lesson as a guide, create a new app where a `student` has many `instructors` through a `course`.

<br>

## Additional Resources

- [Rails has many through association](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)
- [Rails select tag helper](https://apidock.com/rails/ActionView/Helpers/FormTagHelper/select_tag)
