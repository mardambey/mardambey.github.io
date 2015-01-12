---
layout: post
title: From Typo to Tumblr
date: '2012-08-25T21:05:00-04:00'
tags:
- typo
- tumblr
- ruby
tumblr_url: http://www.hisham.cc/post/30207938567/from-typo-to-tumblr
---
I’ve been meaning to update the Typo install that runs this site but never got the chance to do it. I looked around for a bit trying to find a replacement and finally settled on tumblr. As a result of that choice I had to cobble up a way to export posts from Typo 5.0.2 to Tumblr.

After a bit of googling around I ran across a few scripts that people wrote that seemed to work to a certain point then crash. Some were old and some werent compatible with my version of Typo. After a bit more searching and tinkering around I found some Ruby code here that allowed me to publish the Typo database to a Wordpress compatible xml file that I was able to push to Tumblr using their (simpler) V1 API with a bit of Ruby:

require "rubygems"
require "hpricot"
require "net/http"
require "uri"
require "cgi"

def http_get(domain,path,params)
  return Net::HTTP.get(domain, "#{path}?".
    concat(params.collect { |k,v| "#{k}=#{CGI::escape(v.to_s)}" }.join(''&''))) if not params.nil?
  return Net::HTTP.get(domain, path)
end

doc = open("wp.xml") { |f| Hpricot(f) }

(doc/"item").each do |item|
  params = {    
    :email     => "email",
    :password  => "password",
        :type      => "regular",
    :state     => "published",
    :format    => "markdown",
    :date      => item.at(''wp:post_date'').innerHTML,
    :tags      => (item/"category").collect{|category| category.innerHTML}.join(","),
        :title     => item.at(''title'').innerHTML,
    :body      => item.at(''content:encoded'').innerHTML,
  }

  Net::HTTP.post_form(URI.parse("http://www.tumblr.com/api/write"), params)
end


I’ve finally moved away from that old install - one less thing to worry about (I hope!).
