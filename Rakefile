require 'html/proofer'

task :test do
   # build
   sh "bundle exec jekyll build"

   # test
   HTML::Proofer.new("./_site", {
    :href_ignore => ["https://www.linkedin.com/in/bartlettjon"],
    :typhoeus => {
      :headers => { "User-Agent" => "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_5_8; en-US) AppleWebKit/532.8 (KHTML, like Gecko) Chrome/4.0.302.2 Safari/532.8"}
   }}).run

end


