desc "Compile all HAML"
task :default do
  puts %x{cd _src && rake compile:haml}
end

desc "Start Jekyll and Compass watcher"
task :startup do
  begin
    jekyll = fork { exec("jekyll --server --auto --pygments") }
    compass = fork { exec("cd _src && bundle exec compass watch . -c Rakefile") }
  
    Process.wait
  rescue Interrupt
    puts "Caught interrupt. Exiting!"
    Process.kill("HUP", jekyll)
    Process.kill("HUP", compass)
    Process.wait
  end
end