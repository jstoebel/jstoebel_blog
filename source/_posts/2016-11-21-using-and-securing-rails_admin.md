---
layout: post
title: "using (and securing) rails_admin"
permalink: using-and-securing-rails_admin
date: 2016-11-21 09:18:06
comments: true
description: "using (and securing) rails_admin"
keywords: ""
categories:

tags:

---
One expansion to the [EDS Dashobard](https://bitbucket.org/stoebelj/eds_dashboard) is all of the necessary features surrounding assessments. Our team worked diligently on the backend infrastructure this summer, but stopped short at implementing a front end for these features. Specifically we needs a way for users to:

 - Create new `assessments` and `assessment_versions`.
 - Create new `assessment_items` and associate them with `assessment_versions`.
 - Upload `student_scores` for a particular assessment.

Originally the intent was to create a front end in React to allow users to accomplish the above user stories in a single, simple to user interface. While this was a nice idea, other priorities for the dashboard quickly mounted pushing this to the back burner. We found ourselves in a predicament. This was a very important feature, but the time cost seemed undoable given everything else we had to accomplish.

Then I stepped back and reflected on the situation some more. These user stories really only apply to one user: the admin of the app (me). As such, while the functionality is still a high priority, a slick user interface is not. To that end, enter [rails_admin](https://github.com/sferik/rails_admin) a gem for creating a basic CRUD for each of your models. Rails_admin creates an interface based on your existing models. It lets you assign or create associated records on one page, import/export from/to CSV and a whole lot more. Perfect! I had been using MySQL workbench when I needed to peek into production data or make small tweaks, but since it bypasses the model layer, rails_admin is a much better option all around.

## Authorizing

Authorization proved a bit tricky for me. I found something that works for now, but I would prefer a better solution in the future. Regular pages on my app are routed through a a controller that I implement. As such, I am typically able to access the method `current_user` and determine from there if the user should be able to access the resource. For `rails_admin` we define a block in an initializer to determine how authorization should work. I have not yet determined how to access `current_user` inside this block. Fortunately, `current_user` simply looks at the session hash to determine user name. Here's the code:

```
config.authorize_with do
  case Rails.env
  when "production"
    user_name = session[:user]
  when "development"
    user_name = session[:user]
  when "test"
    user_name = request.filtered_parameters["env"]["REMOTE_USER"]
  else
    raise "unknown enviornment: #{Rails.env}"
  end

  user = User.find_by :UserName => user_name
  redirect_to "/access_denied" if [!user.andand.is?("admin") || user.nil?].any?
end
```

## Testing

Since `rails_admin` is not from a controller I own, a controller test is not possible. Instead I setup an integration test:

```
class RailsAdminTest < ActionDispatch::IntegrationTest
  allowed_roles = ["admin"]
  all_roles = Role.all.pluck :RoleName

  test "gets admin" do
    request_admin("admin")
    assert_response :success
  end

  describe "doesn't get admin" do
    (all_roles - allowed_roles + [nil]).each do |r|
      test "as #{r}" do
        request_admin(r)
        assert_redirected_to "/access_denied"
      end
    end
  end

  private
  def request_admin(role_name)
    role = Role.find_by :RoleName => role_name
    user = User.find_by :Roles_idRoles => role.andand.id
    get '/admin', env: { "REMOTE_USER" => user.andand.UserName}
  end

end
```

Typically I use a helper method called `load_session` which loads up the session hash to simulate a logged in user. This approach doesn't seem to work here (for reasons still unknown to me I get an infinite recursion). Instead I had to pass a username directly into the request. I'm not satisfied with this process as it seems like an unnecessary deviation of actual production behavior, but at this point it is the only solution I could work out. Currently, I'm looking for a better one.
