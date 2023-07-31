# terraform_instance
terraform, create instances, launch template, autoscaling group and ALB of that. 
provider "aws" {
  region     = "us-east-1"
  access_key = "AKIA5NYDYLDGALTFFGHE"
  secret_key = "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"
}

resource "aws_instance" "web_server" {
  count         = 2  
  ami           = "ami-05548f9cecf47b442" 
  instance_type = "t2.micro" 

  
  tags = {
    Name = "WebServer"
  }
}

resource "aws_launch_template" "web_server_template" {
  name_prefix             = "WebServer-Template"
  image_id                = "ami-05548f9cecf47b442" 
  instance_type           = "t2.micro"

 
}

resource "aws_autoscaling_group" "web_server_asg" {
  name                      = "web_server_asg"
  max_size                  = 5 
  min_size                  = 2 
  desired_capacity          = 2 
  launch_template {
    id = aws_launch_template.web_server_template.id
    version = "$Latest"
  }

  vpc_zone_identifier = ["subnet-0d71acc8857a3dacb", "subnet-0f4d2e40cfe9ffa50"]
  target_group_arns = [aws_lb_target_group.web_server_lb_target_group.arn]
}

resource "aws_lb" "web_server_lb" {
  name               = "WebServer-LB"
  internal           = false  
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_server_lb_sg.id]
  subnets            = ["subnet-0d71acc8857a3dacb", "subnet-0f4d2e40cfe9ffa50"]  

  tags = {
    Name = "web_server_lb"
  }
}

resource "aws_security_group" "web_server_lb_sg" {
  name_prefix = "web_server_lb_sg"
}

resource "aws_lb_target_group" "web_server_lb_target_group" {
  name     = "WebServer-TG"
  port     = 80
  protocol = "HTTP"
  vpc_id   = "vpc-016202987fdcb712a"  

  health_check {
    path = "/"
  }
}

resource "aws_lb_listener" "web_server_lb_listener" {
  load_balancer_arn = aws_lb.web_server_lb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web_server_lb_target_group.arn
  }
}
