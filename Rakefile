require 'html/proofer'

task :test do
    HTML::Proofer.new("./_site", {
        :href_ignore => [
            '#',
            /mibbit/
        ],
        :typhoeus => {
            headers: { Accept: "html, */*" }
        },
        :verbose => true
    }).run
end
