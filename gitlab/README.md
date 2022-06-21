# [Change gitlab password](https://docs.gitlab.com/ee/security/reset_user_password.html)
 - `gitlab-rails console`
 -     user = User.find_by_username
 -     new_pass = ""
       user.password = new_pass
       user.password_confirmation = new_pass
       
