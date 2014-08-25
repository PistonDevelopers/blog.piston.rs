require 'html/proofer'

task :test do
    HTML::Proofer.new("./_site", {
        :href_ignore => [
            '#',
            /mibbit/
        ],
        :ssl_verifypeer => false,
        :verbose => true
    }).run
end
