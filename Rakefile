require 'html/proofer'

task :test do
   sh "bundle exec jekyll build"
#   HTML::Proofer.new("./_site").run
#
# call with useragent set to workaround linkedin issue https://github.com/gjtorikian/html-proofer/issues/215
   HTML::Proofer.new("./_site", {
    :typhoeus => {
      :headers => { "User-Agent" => "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_5_8; en-US) AppleWebKit/532.8 (KHTML, like Gecko) Chrome/4.0.302.2 Safari/532.8"}
   }}).run

end


