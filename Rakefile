require 'pathname'

def bolt_home
  Pathname(Bundler.definition.specs.find{|x| x.name == "bolt"}.full_gem_path)
end

def rebolt_home
  Pathname(__FILE__).dirname
end

task default: %w[setup]

desc "Run tasks needed to prepare Bolt"
task setup: %w[setup:binstubs setup:modules setup:info setup:check_path]

namespace :setup do
  desc "Install binstubs"
  task :binstubs do
    system("bundle binstubs --path=./vendor/bin --all")
  end

  desc "Install modules with Bolt's Puppetfile"
  task :modules do
    Dir.chdir(bolt_home) do
      system("r10k puppetfile install --verbose --color")
    end
  end

  desc "Confirm whether bolt is on your $PATH"
  task :check_path do
    expected_path = (rebolt_home/"bin/bolt")

    puts
    if(expected_path.exist? && expected_path.executable? && expected_path.symlink?)
      puts "bolt is linked at: #{expected_path}"

      # we need to check the user's real path, not bundler's
      path = ENV["PATH"]
      ENV["PATH"] = ENV["BUNDLER_ORIG_PATH"]

      which = %x[command which which].strip
      actual_path = %x[#{which} bolt].strip
      expected_path = expected_path.to_s

      puts
      if(expected_path == actual_path)
        puts "  #{expected_path} is in your PATH! To run Bolt just type: bolt"
      elsif(actual_path == "")
        puts "  Consider adding #{rebolt_home}/bin to your $PATH to make it easier to run bolt."
        puts "  If you are using bash or zsh adding the following line to your .bashrc or .zshrc should work:"
        puts
        puts "    [[ -x \"#{expected_path}\" ]] && PATH=\"#{rebolt_home}/bin:$PATH\""
      else
        puts '  It appears you already have a different `bolt` in your PATH at: '+actual_path
        puts "  You can run the version we just installed by typing: #{expected_path}"
      end
      puts

      # restore PATH state, just in case any more tasks are run
      ENV["PATH"] = path
    else
      puts "bolt should be linked at #{expected_path}, but it's not there, try running "+'`bundle`'
    end
  end

  desc "Print path to vendored Puppet modules"
  task :info do
    puts
    puts "Modules installed from Puppet Forge can be found at:"
    puts
    puts "  #{bolt_home/"modules"}"
  end
end


desc "Remove modules installed w/ Bolt's Puppetfile"
task clean: %w[clean:modules]

namespace :clean do
  desc "Run (nearly) all clean tasks"
  task all: %w[clean:binstubs clean:modules]

  desc "Remove Bundler binstubs"
  task :binstubs do
    bin_dir = Pathname('.')/"vendor/bin"
    rm_r(bin_dir) if bin_dir.exist?
  end

  desc "Remove modules installed w/ Bolt's Puppetfile"
  task :modules do
    Dir.chdir(bolt_home) do
      puppetfile = rebolt_home/"share/Puppetfile.clean"
      system("r10k puppetfile purge --puppetfile=\"#{puppetfile}\"")
    end
  end

  desc "clean:all, plus reset Bolt's first time run banner."
  task pristine: %w[clean:all clean:first_runs_free]

  task :first_runs_free do
    dotfile = Pathname("~/.puppetlabs/etc/bolt/.first_runs_free").expand_path

    rm(dotfile) if dotfile.exist?
  end
end
