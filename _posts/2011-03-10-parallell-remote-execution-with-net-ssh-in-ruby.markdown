---
layout: post
title: Parallell remote execution with net/ssh in Ruby
date: 10 Mar 2011 17:26:43 UTC
categories: [ruby]
slug: >
  I idle a lot in the "#ruby":irc://irc.freenode.net/#ruby IRC channel
  on Freenode (and "#ruby-lang":irc://irc.freenode.net/#ruby-lang).
  Today, a rather peculiar question was asked by somebody I will hereon
  call Frank…
---

Franks original question — and his example implementation — was a bit confusing. His question, coupled with code, was:

> any suggestions on preserving the order of an array when using threads? i want it to print back the array in the order it was added in.

{% highlight ruby %}
threads = []
IO.readlines('file.txt').each do |line|
  threads << Thread.new(line) do 
    puts line if line =~ /^test/
  end
end

threads.each { |thread| thread.join }

# Output:
# test line 4
# test line 2
# test line 5
# test line 3
# test line 1
{% endhighlight %}

Frank wanted the output in order, meaning he wanted to print *after* the previously spawned thread had printed. After some delirium (I have the flu), I whopped up an example using `#inject`:

{% highlight ruby %}
(1..4).to_a.inject(Thread.new {}) do |t, i|
  Thread.new do
    sleep(rand * 10 % 3) # simulate long-running operation
    puts "Lookup #{i} done"
    t.join # wait for previous thread to finish
    puts "Show result #{i}"
  end
end.join

# Output:
# Lookup 3 done
# Lookup 2 done
# Lookup 1 done
# Show result 1
# Show result 2
# Show result 3
# Lookup 4 done
# Show result 4
{% endhighlight %}

Now that is nice; it works! Frank was happy, and quickly disappeared with a “<q>[…] thanks! im going to try it in a few mins […]</q>”.

## Two hours later, Frank is back!
He’s very excited. He has now written a script that first reads a list of ip addresses from the user, and then goes on to visit each address and running a script on it using [net/ssh](https://github.com/net-ssh/net-ssh). It looked like this:

{% highlight ruby %}
@iplist = []
IO.readlines(ARGV[0]).inject(Thread.new {}) do |t, line|
  label, ip, pass, hostname, ip2 = line.split(/\s+/)
  Thread.new do
    Net::SSH.start("#{ip}", "root", :password => "#{pass}") do |ssh|
      @iplist << ssh.exec!("head -n1 /etc/ips|cut -d : -f1").match(/(\d{1,3}\.+\d{1,3}\.+\d{1,3}\.+\d{1,3})/)
    end
    t.join # wait for previous thread to finish
  end
end.join
puts @iplist
{% endhighlight %}

Now, this is quite different from what Frank originally asked! This is a script that will read a list of hosts from a file, and then visit each host executing the command within `ssh.exec!` on each host. In the end, it will print the results.

Me and Frank discussed his code and why he was doing this for a few minutes. He explained to me that he works as a system administrator, and does not really consider himself a programmer. Frank is too hard on himself.

Either way, I eventually began to see the code as a puzzle: could I improve the code, making it nicer and useful in other cases — and possibly even help Frank even more?

The answer to that is yes! Ten minutes later, I showed Frank what I came up with:

{% highlight ruby %}
require 'net/ssh'

ips = $stdin.readlines.map do |line|
  label, host, user, command = line.split(' ', 4)
  [label, Thread.new do
    output = nil
    Net::SSH.start(host, user) { |ssh| output = ssh.exec!(command) }
    output.to_s
  end, line]
end

ips.each do |(label, thread, _)|
  puts "[#{label}]"
  thread.value.each_line.map do |line|
    puts "  " + line
  end
end
{% endhighlight %}

Executing this code like so `ruby script.rb < instructions.txt`, with instructions.txt containing this:

<div class="highlight"><pre>superman 127.0.0.1 Kim whoami
database 127.0.0.1 Kim sleep 1 &amp;&amp; echo "Database says Hello!"
localhost 127.0.0.1 Kim date</pre></div>

Will give this output:
<div class="highlight"><pre>[superman]
  Kim
[database]
  Database says Hello!
[localhost]
  Thu Mar 10 23:31:59 CET 2011</pre></div>

Frank was happy, I was happy. Me and Frank talked for another twenty minutes, and then parted ways. Goodbye Frank, it was nice talking to you today!