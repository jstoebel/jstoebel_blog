---
layout: post
title: "A Lesson in Refactoring Complex Classes in Ruby"
permalink: a-lesson-in-refactoring-complex-classes-in-ruby
date: 2018-08-30 10:51:05
comments: true
description: "A Lesson in Refactoring Complex Classes in Ruby"
keywords: ""
categories:

tags:

---

I have always loved the act of refactoring, especially in Ruby. Its a fun puzzle to solve and the reward at the end is code that's easier to read and maintain.

One thing I've spent a time refactoring is when a classes' private methods become so complex, that it becomes hard to test and reason about. What started as a simple class with a simple interface begins to feel more like an iceberg: looks simple from the outside, but tons more under the surface. We might be tempted to not worry about a growing collection of private methods. "So long as the right output happens in the end, we're fine!" we say to out selves. The issue there is with testability: If I want to consider all of the corner cases and gotchas that could go wrong while "under the surface", I need to set up those corner cases using the public interface. Suddenly my unit tests on this classes start to feel like more like integration tests: give an input to one part of the system and observe the reaction in another part of the system. When different parts of a class start to feel like they aren't one things but two (or even three, four or more), that's when I know its time to start refactoring.

Let's consider a class that's responsible for getting some data from a remote server. Its going to run a command over ssh which will result in a file on the remote machine being created. Depnding on the reuslt of that ssh command the class then `scp`s the file down and then removes the remote file over ssh to clean things up. I'm going to omit the implementation of the methods since its not important for this example. 

```
class CopyDb

  def initialize
  end

  def perform
    dump_output =_ssh 'rake db:backup'
    raise unless output_ok? dump_output
    _scp_file
    delete_output = _ssh 'rm filename.txt'
    raise unless output_ok? delete_output
  end

  private

  # issues command over ssh to the configured server, returns the remote output
  def _ssh(command)
    ...
  end

  # parses output from a remote ssh session, returns true if exited with code 0
  def output_ok?(remote_output)
    ...
  end

  # parses remote output to determine the created file name and returns that name
  def remote_file_name(remote_output)
  end

  # pull down the expected file, raise an error if there was a problem
  def _scp_file
    ...
  end

end
```

This of course is a very simplified version of what this class would actually look like, but as I was working on it, and trying to write the tests things started getting hairy. I of course didn't want to actually have my tests hit the remote server, but to test all of the possible outcomes I need to set up the appropriate stubbed output in each test. This quickly started to feel tedious. I realized that I was trying to represent two things (the copying of the database and the remote server) in one class. What if I set up a new class called `Server` that acted as a collaborator class and handled all of the interactions with the remote server? It would provide two public methods `ssh` and `scp` and would return a `Struct` represnting the remote output and if it exited with out error:

```
# frozen_string_literal: true

require 'ostruct'

# class that encapsulates all interactions with the remote server
class Server
  # username(string): the remote username for the server
  # host(string): the host name for the server
  # project_root(string): the name of the project directory where remote operations will be performed
  def initialize(username, host, project_name)
    @username = username
    @host = host
    @project_name = project_name
    @project_root = "/web/project/#{project_name}/current"
  end

  # perform a command over ssh at the @project_root directory
  # cmd(string): the remote command to execute
  # returns output of the command plus the exit code on the remote process
  def ssh(cmd)
    full_cmd = "cd #{@project_root} && #{cmd}; echo $?"
    raw_output = _ssh full_cmd
    _parsed_results raw_output
  end

  # retrieve a file via scp command located at @project_root
  # and place it (same name) in pwd
  # raises error when failed
  # returns nil
  def scp(file_name)
    full_file_path = "#{@project_root}/#{file_name}"
    _scp file_name, full_file_path
  end

  private

  # handles the ssh call
  def _ssh(full_command)
    `ssh #{@username}@#{@host} '#{full_command}' 2>&1`
  end

  # output (string): the output from the remote server
  def _parsed_results(output)
    remote_command_ok = output.split("\n").last == '0'
    OpenStruct.new output: output, success: remote_command_ok
  end

  def _scp(file_name, full_file_path)
    Net::SCP.download!(
      @host,
      @username,
      full_file_path,
      "./#{file_name}"
    )
  end
end
```

Now, we have a better seperation of concerns. The `CopyDb` class handles the processing of information and a collaborating instance of the `Server` class is how we interact with the remote server. The biggest pay off for me was with testability. Now I can write unit tests to ensure a `Server` can do its job correctly, and then when I want to test `CopyDb` I only need to ensure that the correct inputs are passed to the collaborating `Server`. Better seperation of concerns == refactoring win!