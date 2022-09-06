NullMailer /etc/nullmailer/remotes :
smtp.server.com  smtp --port=465 --user=sender@domain.com --pass=mypass --ssl

AND

sendmail myrecipient@domain.com
CTRL+D to send
(Config for sender in etc/nullmailer/allmailfrom)

OR

mail -aFrom:'The Name<sender@domain.com>' -s "My Subject" myrecipient@domain.com
CTRL+D to send
