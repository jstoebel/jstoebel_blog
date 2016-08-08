---
layout: post
title: Creating a new exercise on CodeWorkout-Part 2
date: '2016-02-06T21:57:17+00:00'
permalink: creating-a-new-exercise-on-codeworkout-part-2
---
Note: this post is a continuation of this post
###TL;DR
CodeWorkout does not yet have a UI for faculty to add a new exercise. In this blog post I will walk you though how to create a new coding exercise from the rails console. The purpose here is more to understand how it works. Let's not waste time entering questions this way when we actually go to contribute questions!</p>
To get started, make sure you are in the root of the app and typerails console  or simple `rails c`  for short. This will bring up the rails console which allows us to issue Ruby command against our Rails environment.
###Exercise
Records in the exercises table of the database are represented by ActiveRecord as a Ruby class called <code>Exercise</code>. Let's create a new one now:

    irb(main):001:0&gt; ex = Exercise.new
    => #Exercise id: nil, question_type: nil, current_version_id: nil, created_at: nil, updated_at: nil, versions: nil, exercise_family_id: nil, name: nil, is_public: false, experience: nil, irt_data_id: nil, external_id: nil>

We've just created a new Exercise instance. Its not in the database yet (how could it, it has no attributes including an id). What should we assign to these fields?


- **id**: This is the primary key of the record, meaning that it is how we reference the record uniquely. It is also auto incrementing, meaning that the database will simply give it an arbitrary number when we save it to the database. We don&#8217;t need to put anything here.
- **question_type:** CodeWorkout allows for several types of questions such as coding and multiple choice. If we look in the exercise model we see the following.


        Q_MC = 1
        Q_CODING = 2
        Q_BLANKS = 3
        TYPE_NAMES = {
        Q_MC = 'Multiple Choice Question'
        Q_CODING = 'Coding Question'
        Q_BLANKS ='Fill in the blanks'
        }

We want to make a coding question so let's type `ex.question_type = 2`.

- **current\_version\_id** Each exercise is allowed to have multiple versions which are kept in a table exercise_versions. The most current version
is referenced here. We'll leave it blank for now, but before we can commit this record we need to create an exercise_version and then reference it here.
- **created\_at and updated\_at **
These fields simply represent when the record was made and last updated. The database will handle filling and updating the time stamps here. We can leave them as nil.
*   **versions** How many versions of this exercise there are. Let’s enter `ex.versions = 1`*   **exercise_family_id** Each exercise can belong to a family of exercises, but doesn’t have to. Looking at the records in exercise_familes table (`ExerciseFamily.all`) we see no records. Let’s skip this for the time being.*   **name** That’s an easy one: simply the name of the exercise. Let’s call it `ex.name = "Hello, Python!"`*   **is_public**  
    If this exercise should be made public. We can type `ex.is_public = true` but the record will actually default to this when we save it. How do we know? Looking at the model file `exercise.rb` we see the following before_validation :set_default*   This means that before a record is validated (and all records must be first validated before the are saved to the database, rails should run the method set_defaults). Scrolling to the bottom of the file we see this method definition: def set_defaults # Update current_version if necessary if !self.current_version self.current_version = self.exercise_versions.first endself.question_type ||= (current_version && current_version.prompts.first) ? current_version.question_type : Q_MC self.name ||= ” self.is_public ||= true self.experience ||= 10 end On line 212 we see that rails will set is_public to true if the value is currently nil (that’s what ||= means; assign it if its nil)*   **experience** How much experience should the user gain if they answer correctly? Defaults to 10.*   **rt_data** I’m not sure what this is yet, but I think it has something to do with the model irt_data.rb. I’m skipping it for now and will come back to it later*   **external_id** Again, not sure what it is just yet. Skipping for the moment.

Go ahead and save the exercise by type `ex.save` If `true` is returned, the record has been successfully saved to the database.

## ExerciseVersion

We’re almost done making an `Exercise` record. Next we need to make an ExerciseVersion and associate it with the exercise. Let’s jump over to that that model in `exercise_version.rb`. Let’s also make a new instance. In the console type  
 
    irb(main):016:0> ex_ver = ExerciseVersion.new  
    => #<ExerciseVersion id: nil, stem_id: nil, created_at: nil, updated_at: nil,     exercise_id: nil, version: nil, creator_id: nil, irt_data_id: nil>`

Let’s walk through the fields of this record

*   **id** Auto-incrementing primary key. Leave it alone!
*   **stem_id** If we peek in the stem.rb file we see the description: “Represents the stem, or preamble, for an exercise.” Let’s skip it for now just for the sake of simplicity.
*   **created_at and updated_at** Skip! (see above)
*   **exercise_id** This is the id referencing the exercise record.  
    Assign this version to our exercise by sticking its id into this field. `ex_ver.exercise_id = ex.id`
*   **version** Which version is this? Its 1 so `ex_ver = 1`
*   **creator_id and irt_data_id** Skipping these for the moment!

We’re ready to save this version. Type `ex_ver.save`

## Prompt and CodingPrompt

Our exercise is chugging along but its missing some content. Namely, the actual question we want to ask, along with any code to give the user and some other key attributes. For coding questions, those attributes are stored in `CodingPrompt` and its psudo base class `Prompt`. Its a little complicated but the basic idea is that all exercises, regardless of their type share some common attributes. Those attributes are stored in the Prompt class. However a coding exercise has specific attributes that don’t make sense for a multiple choice question, these attributes are found in the `CodingPrompt` exercise. CodeWorkout uses a gem called acts_as which allows attributes of the Prompt class be accessed directly by the `CodingPrompt` class. Let’s make a new `CodingPrompt` record and associate it with out `ExerciseVersion`.


    irb(main):041:0> cd_pmt = CodingPrompt.new   
    => #<CodingPrompt id: nil, created_at: nil, updated_at: nil, class_name: nil, wrapper_code: nil, test_script: nil, method_name: nil, starter_code: nil>  


`CodingPrompt` doesn’t look like much but remember that it also knows about all of the attributes that `Prompt` knows about. Let’s look at both of them.

*   **id** Auto-incrementing PK. Leave it alone!
*   **exercise_version_id** The `ExerciseVersion` this prompt will reference.  
    `cd_pmt.exercise_version_id = ex_ver.id`
*   **question** This the simply the question to ask the user  
    `cd_pmt.question = "print the phrase 'Hello, World!'" to the screen`
*   **position** a question can have multiple prompts. For simplicity, ours will just have one so leave it blank.
*   **feedback** Feedback that gets displayed to the user. Leave blank for this example!
*   **actable_id** skip! It will be automatically assigned.
*   **actable_type** What kind of prompt is this prompt “acting as”. For this one I looked at an existing CodingPrompt.

        irb(main):062:0> CodingPrompt.first.actable_type  
        SQL (0.7ms) SELECT "coding_prompts"."id" AS t0_r0, "coding_prompts"."created_at" AS t0_r1, "coding_prompts"."updated_at" AS t0_r2, "coding_prompts"."class_name" AS t0_r3, "coding_prompts"."wrapper_code" AS t0_r4, "coding_prompts"."test_script" AS t0_r5, "coding_prompts"."method_name" AS t0_r6, "coding_prompts"."starter_code" AS t0_r7, "prompts"."id" AS t1_r0, "prompts"."exercise_version_id" AS t1_r1, "prompts"."question" AS t1_r2, "prompts"."position" AS t1_r3, "prompts"."feedback" AS t1_r4, "prompts"."created_at" AS t1_r5, "prompts"."updated_at" AS t1_r6, "prompts"."actable_id" AS t1_r7, "prompts"."actable_type" AS t1_r8, "prompts"."irt_data_id" AS t1_r9 FROM "coding_prompts" LEFT OUTER JOIN "prompts" ON "prompts"."actable_id" = "coding_prompts"."id" AND "prompts"."actable_type" = 'CodingPrompt' ORDER BY "coding_prompts"."id" ASC LIMIT 1  
        => "CodingPrompt"

    We should call this prompt a `CodingPrompt` type `cd_pmt.actable_type = "CodingPrompt"

*   **irt_data_id** Skip for this example.
*   **class_name** If we wanted our code to include a class name we would put it here. Since we are making a simple Hello, World in python we don’t need any classes. Leave blank!
*   **wrapper_code**  
    Haven’t figured out what this is for yet, but it is required. Just give it something random like `cd_pmt.wrapper_code = "spamspamspam"`
*   **test_script**  
    The code that should run to test that the user entered the right answer. We need to enter something, but let’s not worry about entering a valid test. Just enter `cd_pmt.test_script = "testing123"`
*   **method_name** If we are assigning a method to this assignment what should it be called? We can skip cause this is Python!
*   **starter_code** If we want to start the user off with some code to work from put it here. I think the user can figure it out on their own, so let’s leave this blank.

We should be ready to save our record now. Type `cd_pmt.save`. And by the way. If you ever type `.save` and get `false` it means the record wasn’t saved. To see why type `object.errors.full_messages`. This will display all of the reasons why your record can’t be saved. Correct them and try to save again.

## Test it out!

Now let’s fire up our rails server (`rails s`) and visit `[http://localhost:3000](http://localhost:3000)`. Login as admin (credentials provided in the README.doc) and visit Gym -> exercises -> browse all. Our exercise should be a the bottom!
