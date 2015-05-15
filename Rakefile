require 'net/http'
require 'json'
require 'timeout'
require 'nokogiri'

task :default => [:test_fresh_install]
ENV["DOCKER_HOST"]||="tcp://localhost:4243"


task :test_fresh_install do
  ["ubuntu","centos"].each do |flavour|
    test_fresh_installer flavour
  end
end


#task :test_upgrade_install do
#  ["ubuntu"].each do |flavour|
#    prepare_old_go_base_image flavour
#    test_server_upgrade_to_latest flavour
#    test_agent_upgrade_to_latest flavour
#  end
#end



task :test_fresh_postgres_install do
  ["ubuntu","centos"].each do |flavour|
    test_fresh_installer_with_postgres flavour
  end
end



def test_fresh_installer flavour
  begin
    server_name = "go"
    image= "go-server-fresh-install-#{flavour}"
    dockerfile_path = "fresh-install/#{flavour}"

    go_version = get_go_version_of_installer_to_test
    set_go_installer_version_in_dockerfile dockerfile_path,go_version

    build_image dockerfile_path, image

    container_id = start_container image,"container_id"
    pipeline_name = "test"
    wait_for_server server_name
    wait_for_go_agent
    create_pipeline pipeline_name
    wait_for_pipeline_to_go_green server_name, pipeline_name,"1"
  ensure
    stop_container container_id
    cleanup "container_id"

    cleanup_go_installer_version_from_dockerfile dockerfile_path
    cleanup "go_version.txt"
  end
end



def prepare_old_go_base_image flavour
  begin

    pipeline_name = "test"
    container_id = nil

    server_name = "go"
    image = "initial-old-go-#{flavour}"
    dockerfile_path = "upgrade/#{flavour}/earlier"


    build_image dockerfile_path, image

    #container_id = start_container image,"container_id"
    #
    #wait_for_server server_name
    #wait_for_go_agent
    #create_pipeline pipeline_name
    #wait_for_pipeline_to_go_green server_name, pipeline_name,"1"
    #
    #sleep 20
    #run_system_command("docker commit #{container_id} old-go-#{flavour}")

  ensure
    #stop_container container_id_go
    #cleanup "container_id_go"
  end
end


def test_server_upgrade_to_latest flavour
  begin

    #### Upgrade only server first ########

    server_name = "go"
    pipeline_name = "test"
    container_id = nil

    image = "initial-go-server-upgrade-to-latest-#{flavour}"
    dockerfile_path = "upgrade/#{flavour}/latest"

    go_version = get_go_version_of_installer_to_test
    set_go_installer_version_in_dockerfile dockerfile_path,go_version


    build_image dockerfile_path, image

    container_id = start_container image, "#{flavour}_container_id"

    wait_for_server server_name
    wait_for_go_agent
    trigger_go_pipeline pipeline_name
    wait_for_pipeline_to_go_green server_name, pipeline_name,"2"

    sleep 20
    run_system_command("docker commit #{container_id} go-server-upgrade-to-latest-#{flavour}")


  ensure
    stop_container container_id
    cleanup "#{flavour}_container_id"

    cleanup_go_installer_version_from_dockerfile dockerfile_path
    cleanup "go_version.txt"
    end

end


def test_agent_upgrade_to_latest flavour
  begin

    #### Upgrade agent also to latest ########

    server_name = "go"
    pipeline_name = "test"
    container_id = nil

    image = "go-server-and-agent-upgrade-to-latest-#{flavour}"
    dockerfile_path = "upgrade/#{flavour}/latest-agent"

    go_version = get_go_version_of_installer_to_test
    set_go_installer_version_in_dockerfile dockerfile_path,go_version


    build_image dockerfile_path, image

    container_id = start_container image, "#{flavour}_container_id"

    wait_for_server server_name
    wait_for_go_agent
    trigger_go_pipeline pipeline_name
    wait_for_pipeline_to_go_green server_name, pipeline_name,"3"


  ensure
    stop_container container_id
    cleanup "#{flavour}_container_id"

    cleanup_go_installer_version_from_dockerfile dockerfile_path
    cleanup "go_version.txt"
  end

end




def test_fresh_installer_with_postgres flavour
  begin
    server_name = "go"
    pipeline_name = "test"
    dockerfile_path= "fresh-install-postgres/#{flavour}"

    build_image "postgres-with-password-auth-setup", "postgres_image"

    container_id_postgres = start_container_without_binding "postgres_image","container_id_postgres"
    postgres_ip = get_server_ip_for_container container_id_postgres
    set_server_host_ip_in_dockerfile dockerfile_path, postgres_ip

    go_image = "go-server-fresh-install-postgres-#{flavour}"

    version = get_go_version_of_installer_and_postgres_version_of_jar_to_test
    set_postgres_jar_version_in_dockerfile dockerfile_path,version[:postgres]
    set_go_installer_version_in_dockerfile dockerfile_path,version[:go]

    build_image dockerfile_path, go_image

    container_id = start_container go_image, "container_id"

    wait_for_server server_name
    wait_for_go_agent
    create_pipeline pipeline_name
    wait_for_pipeline_to_go_green server_name,pipeline_name,"1"

   ensure
    stop_container container_id
    stop_container container_id_postgres

    cleanup "container_id"
    cleanup "container_id_postgres"
    cleanup "ip.txt"
    cleanup "go_and_postgres_version.txt"

    cleanup_server_ip_from_dockerfile dockerfile_path
    cleanup_postgres_jar_version_from_dockerfile dockerfile_path
    cleanup_go_installer_version_from_dockerfile dockerfile_path

  end
end



############

def stop_container id
  if (container_exists(id)) then
   run_system_command("docker stop #{id}") if id
   run_system_command("docker kill #{id}") if id
   run_system_command("docker rm -f #{id}") if id
   puts "Stopped container #{id}"
  end
end


def wait_for_pipeline_to_go_green server_name, pipeline_name,label
  activity = nil
  lastBuildStatus = nil
  buildLabel = nil
  begin
    timeout(400) do
      response = nil
      while(true) do
        response = get_request("http://localhost:8153/#{server_name}/cctray.xml").body
        dom = Nokogiri::XML(response)
        pipeline = dom.at_xpath("//Projects/Project[@name='#{pipeline_name} :: defaultStage']")
        if pipeline then
          lastBuildStatus = pipeline.attr("lastBuildStatus")
          activity = pipeline.attr("activity")
          buildLabel=pipeline.attr("lastBuildLabel")
          if lastBuildStatus == "Success" and activity == "Sleeping" and buildLabel==label then
            puts "Pipeline \'#{pipeline_name}\' with label \'#{label}\' went green. Yay! :)"
            break
          end
        end
      end
    end
  end
  rescue Timeout::Error
    raise "Pipeline was not built successfully, current build status is: #{lastBuildStatus}, activity is: #{activity} and build label is: #{buildLabel}"
end



def get_request url
  uri = URI(url)
  response = nil
  request = Net::HTTP::Get.new(uri.path)
  response = Net::HTTP.start(uri.host,uri.port) do |http|
    http.request(request)
  end
  response
end

def create_pipeline pipeline_name
  url = "http://localhost:8153/go/tab/admin/pipelines/#{pipeline_name}.json"
  uri = URI(url)
  request = Net::HTTP::Post.new(uri.path)
  request.set_form_data({"scm" => "git", "url" => "https://github.com/agoyal-git/testrepo.git", "builder"=> "exec", "command" => "ls"})
  response = Net::HTTP.start(uri.host,uri.port) do |http|
    http.request(request)
  end
  puts "Successfully created pipeline" if response.is_a?(Net::HTTPCreated)
  raise "Pipeline creation failed with error: #{response.body}" unless response.is_a?(Net::HTTPCreated)
end

def wait_for_server server_name
  begin
    Timeout::timeout(240) do
      while(true) do
        begin
          if (server_name=="go") then
            response = get_request(URI.encode("http://localhost:8153/#{server_name}/home"))
            response_latest = get_request(URI.encode("http://localhost:8153/#{server_name}/api/support"))
          end
          if (server_name=="cruise") then
            response = get_request(URI.encode("http://localhost:8153/#{server_name}/pipelines"))
          end
          if (response.is_a?(Net::HTTPOK) || response_latest.is_a?(Net::HTTPOK)) then
            puts "#{server_name} server is up!"
            break;
          else
            raise "Request to Go server failed with error : #{response.body}"
          end
        rescue 
          #ignore eof errors as go server might not have started.
        end
      end
    end
  rescue Timeout::Error
    raise "#{server_name} server is down!"
  end

end

def wait_for_go_agent
  begin
    Timeout::timeout(180) do
      while(true)
        agents = JSON.parse(get_request("http://localhost:8153/go/api/agents").body)
        active_agents = agents.select {|agent| agent["status"] == "Idle" }
        if(active_agents.any?) then
          puts "Agent is up"
          break
        end
      end
    end
  rescue Timeout::Error
    raise "Agent is down!"
  end
end

def run_system_command(cmd)
  result = system(cmd)
  raise "#{cmd} failed!" unless result
  result
end

def cleanup file_name
  File.delete "#{file_name}" if File.exist? "#{file_name}"
end



def trigger_go_pipeline pipeline_name
  url = "http://localhost:8153/go/api/pipelines/#{pipeline_name}/schedule"
  uri = URI(url)
  request = Net::HTTP::Post.new(uri.path)
  response = Net::HTTP.start(uri.host,uri.port) do |http|
    http.request(request)
  end
  puts "Successfully triggered pipeline" if response.is_a?(Net::HTTPAccepted)
  raise "Pipeline trigger failed with error: #{response.body}" unless response.is_a?(Net::HTTPAccepted)
end



def get_server_ip_for_container container_id
  run_system_command("docker inspect #{container_id} | grep IPAddress > ip.txt")
  return File.read("ip.txt").gsub(/\"IPAddress\":\s+\"(\d+\.\d+\.\d+.\d+)\"[,]$/,'\1').strip
end


def build_image dockerfile_path , image_name
  Dir.chdir(dockerfile_path) do
    run_system_command("docker build -t #{image_name} .")
  end
end



def set_server_host_ip_in_dockerfile dockerfile_path ,host_ip
  Dir.chdir(dockerfile_path) do
    #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/server_ip/, "#{host_ip}")})
    original_dockerfile = File.read("Dockerfile")
    new_dockerfile = original_dockerfile.gsub(/server_ip/, "#{host_ip}")
    File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}
  end
end


def start_container image, file_name
  run_system_command("docker run -d --cidfile=#{file_name} -p 8153:8153 --privileged #{image}")
  return File.read("#{file_name}")
  end



def start_container_without_binding image, file_name
  run_system_command("docker run -d --cidfile=#{file_name} #{image}")
  return File.read("#{file_name}")
end



def cleanup_server_ip_from_dockerfile dockerfile_path
 Dir.chdir(dockerfile_path) do
   #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/\d+\.\d+\.\d+\.\d+/, 'server_ip')})
   original_dockerfile = File.read("Dockerfile")
   new_dockerfile = original_dockerfile.gsub(/\d+\.\d+\.\d+\.\d+/, 'server_ip')
   File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}
 end
end


def container_exists container_id
    container_exists = false
    unless container_id.nil?
      docker_host=ENV["DOCKER_HOST"].gsub(/tcp/,'http')
      response= get_request("#{docker_host}/containers/#{container_id}/json")
      if response.is_a?(Net::HTTPOK) then
        puts "container #{container_id} exists"
        container_exists = true
      end
    end
    return container_exists
end


def get_go_version_of_installer_to_test
  run_system_command("cat upstream/console.log | grep \"Determining version of installers to be:\" > go_version.txt")
  return File.read("go_version.txt").gsub(/.*Determining version of installers to be:\s((\d+\.){2}(\d+)-(\d+){4})$/,'\1').strip
end



def set_go_installer_version_in_dockerfile dockerfile_path ,go_version
  Dir.chdir(dockerfile_path) do
    #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/\[go-version-under-test\]/, "#{go_version}")})
    original_dockerfile = File.read("Dockerfile")
    new_dockerfile = original_dockerfile.gsub(/\[go-version-under-test\]/, "#{go_version}")
    File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}
  end
end


def cleanup_go_installer_version_from_dockerfile dockerfile_path
  Dir.chdir(dockerfile_path) do
    #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/(\d+){2}\.\d+\.\d+-(\d+){4}/, '[go-version-under-test]')})
    original_dockerfile = File.read("Dockerfile")
    new_dockerfile = original_dockerfile.gsub(/(\d+){2}\.\d+\.\d+-(\d+){4}/, '[go-version-under-test]')
    File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}  end
end


def get_go_version_of_installer_and_postgres_version_of_jar_to_test
  version = Hash.new
  run_system_command("cat upstream-postgres/console.log | grep \"Finished copying to: go-postgresql\" > go_and_postgres_version.txt")
  postgres_version = File.read("go_and_postgres_version.txt").gsub(/.*Finished copying to: go-postgresql-((\d+\.){2}(\d+)-(\d+))\.jar.*/,'\1').strip
  go_version = File.read("go_and_postgres_version.txt").gsub(/.*Finished copying to: go-postgresql-[\d.\-]+\.jar in ((\d+\.){2}(\d+)-(\d+){4})$/,'\1').strip
  version[:postgres] = postgres_version
  version[:go] = go_version
  return version
end


def set_postgres_jar_version_in_dockerfile dockerfile_path ,postgres_version
  Dir.chdir(dockerfile_path) do
    #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/\[postgres-version-under-test\]/, "#{postgres_version}")})
    original_dockerfile = File.read("Dockerfile")
    new_dockerfile = original_dockerfile.gsub(/\[postgres-version-under-test\]/, "#{postgres_version}")
    File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}
  end
end


def cleanup_postgres_jar_version_from_dockerfile dockerfile_path
  Dir.chdir(dockerfile_path) do
    #IO.write( "Dockerfile" , File.open("Dockerfile") {|f| f.read.gsub(/(\d+){2}\.\d+\.\d+-(\d+)\.jar/, '[postgres-version-under-test].jar')})
    original_dockerfile = File.read("Dockerfile")
    new_dockerfile = original_dockerfile.gsub(/(\d+){2}\.\d+\.\d+-(\d+)\.jar/, '[postgres-version-under-test].jar')
    File.open("Dockerfile", "w") {|f| f.write(new_dockerfile)}
  end
end

