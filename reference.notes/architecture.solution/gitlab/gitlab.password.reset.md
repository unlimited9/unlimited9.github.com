# gitlab management

## password reset  

#### forgot your password  
with e-mail 

#### gitlab-rails console  
```
root@172:/# gitlab-rails console
-------------------------------------------------------------------------------------
 GitLab:       12.0.2 (1a9fd38a4ca)
 GitLab Shell: 9.3.0
 PostgreSQL:   9.6.11
-------------------------------------------------------------------------------------
Loading production environment (Rails 5.1.7)
irb(main):002:0> user = User.find_by(username: 'user01')
=> #<User id:16 @user01>
irb(main):006:0> user.password = 'user011234'
=> "user011234"
irb(main):007:0> user.password_confirmation = 'user011234'
=> "user011234"
irb(main):008:0> user.save
Enqueued ActionMailer::DeliveryJob (Job ID: 4bc12aee-ca79-4e72-b2e0-bbe34c1ede76) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", #<GlobalID:0x00007f95f2e870f8 @uri=#<URI::GID gid://gitlab/User/16>>
=> true
irb(main):009:0> 
```

## appendix  

#### reference site  

+ gitlab 패스워드 리셋하기 - Voyager of Linux  
https://linux.systemv.pe.kr/gitlab-패스워드-리셋하기/  



