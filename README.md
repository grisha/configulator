# Configulator : Generate Config Files

[![Build Status](https://travis-ci.org/bmuller/configulator.svg)](https://travis-ci.org/bmuller/configulator)
 
Configurator allows you to maintain a single file with configuration values that can be used to generate other config files based on templates.

For instance, if you need to generate all of the config files for a Hadoop cluster and the files vary in type (xml, json, yaml, etc) but all rely on the same few configuration values, you can use configulator to generate those files.

Configulator is a tool that takes a single master input config file and some number of template files as input and then outputs the template files with the correct configuration values substituted. Additionally, there is a way of specifying default values, and then configulator will override on the necessary ones per environment.

See [this blog post](http://findingscience.com/linux/sysadmin/ruby/2010/10/27/config-template-class.html) for the background on why this can be useful.

## Installation

Add this line to your application's Gemfile:

    gem 'configulator'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install configulator

## Usage

Configulator can accept a master config file either in json or yaml.  Here's an example config file:

    mysql_host: 127.0.0.1
    mysql_port: 5678
    db_host: <%= mysql_host %>
    db_port: <%= mysql_port %>
    current_time: <%= Time.now %>

It specifies a mysql host and port, as well as a general db host and port that will simply use the values for mysql.  Then, given an example config file that's nasty and written in XML:

    <xml><config><host><%= db_host %></host></config></xml>

We can now convert the above file into one with db_host filled in either using code or the configulator command.  Here's what the code looks like (assuming our master config is master.yml and our xml is xmlisbad.xml.tmpl):

    require 'configulator'
    converted = Configulator.from_yaml_file('master.yml').convert_file('xmlisbad.xml')
    File.write 'xmlisbad.xml', converted

You can also access specific variables in the ruby code:

    puts converted.db_host # => 127.0.0.1

Using the configulator command instead looks like this:

    configulator master.yml xmlisbad.xml.tmpl > xmlisbad.xml

## Environments

You have the option of setting default values in your master config file and then separate environment sections beneath that.  For instance, here's a yaml config file with default values for some keys:

    default:
        debug: INFO
        db_host: <%= mysql_host %>
        db_port: <%= mysql_port %>
    production:
        mysql_host: 1.2.3.4
        mysql_port: 5678
    development:
        mysql_host: 127.0.0.1
        mysql_port: 5678

We can spit out the production values like this (just passing in an extra argument to from_yaml_file):

    require 'configulator'
    converted = Configulator.from_yaml_file('master.yml', 'production').convert_file('xmlisbad.xml')
    File.write 'xmlisbad.xml', converted

When db_host is used in the xml file, configulator will look for it first in the production section and then in the default (where it is found, then repeats process for mysql_host).
