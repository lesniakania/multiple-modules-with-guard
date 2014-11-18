[['.', ''], ['moduleA', 'moduleA/']].each do |dir, prefix|
  guard :rspec,
    chdir: dir,
    cmd: "cd #{dir} && bundle exec spring rspec",
    failed_mode: :keep,
    spec_paths: ['spec'] do

    watch(%r{^#{prefix}(.+)\.rb$})
    watch(%r{^#{prefix}lib/(.+)\.rb$})     { |m| "spec/lib/#{m[1]}_spec.rb" }
    watch("#{prefix}spec/spec_helper.rb")  { "spec" }

    # Rails example
    watch(%r{^#{prefix}app/(.+)\.rb$})                           { |m| "spec/#{m[1]}_spec.rb" }
    watch(%r{^#{prefix}(.+)\.rb$})                               { |m| "spec/#{m[1]}_spec.rb" }
    watch(%r{^#{prefix}app/(.*)(\.erb|\.haml|\.slim)$})          { |m| "spec/#{m[1]}#{m[2]}_spec.rb" }
    watch(%r{^#{prefix}app/controllers/(.+)_(controller)\.rb$})  { |m| ["spec/routing/#{m[1]}_routing_spec.rb", "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb", "spec/acceptance/#{m[1]}_spec.rb"] }
    watch(%r{^#{prefix}spec/support/(.+)\.rb$})                  { "spec" }
    watch("#{prefix}config/routes.rb")                           { "spec/routing" }
    watch("#{prefix}app/controllers/application_controller.rb")  { "spec/controllers" }
    watch("#{prefix}spec/rails_helper.rb")                       { "spec" }

    # Capybara features specs
    watch(%r{^#{prefix}app/views/(.+)/.*\.(erb|haml|slim)$})     { |m| "spec/features/#{m[1]}_spec.rb" }

    # Turnip features and steps
    watch(%r{^#{prefix}spec/acceptance/(.+)\.feature$})
    watch(%r{^#{prefix}spec/acceptance/steps/(.+)_steps\.rb$})   { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'spec/acceptance' }
  end


  guard 'jasmine', :server => :webrick, :server_mount => '/specs' do
    watch(%r{#{prefix}spec/javascripts/spec\.(js\.coffee|js|coffee)$}) { 'spec/javascripts' }
    watch(%r{#{prefix}spec/javascripts/.+_spec\.(js\.coffee|js|coffee)$})
    watch(%r{#{prefix}spec/javascripts/fixtures/.+$})
    watch(%r{#{prefix}app/assets/javascripts/(.+?)\.(js\.coffee|js|coffee)(?:\.\w+)*$}) { |m| "spec/javascripts/#{ m[1] }_spec.#{ m[2] }" }
  end
end

