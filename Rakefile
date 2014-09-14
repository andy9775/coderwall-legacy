require File.expand_path('../config/application', __FILE__)
require 'rake'

Coderwall::Application.load_tasks

task default: :spec


namespace :team do
  task migrate: :environment do
    TeamMigratorBatchJob.new.perform
  end

  #
  # IMPORTANT: pending_join_requests is a STRING array in Postgres but an INTEGER array in MongoDB.
  # IMPORTANT: pending_join_requests is an array of User#id values
  #

  def neq(attr, pg, mongo, fail_if_neq=true)
    left =  pg.send(attr)
    right = mongo.send(attr)

    if left != right
      puts "#{attr}   #{left} != #{right}  (#{pg.id}|#{mongo.id})"

      if fail_if_neq
        require 'pry'; binding.pry
      end
    end
  end

  task verify: :environment do
    PgTeam.find_each(batch_size: 100) do |pg_team|
      begin
        mongo_id = pg_team.mongo_id
        mongo_team = Team.find(mongo_id)


        %i(updated_at median score total slug mean pending_join_requests).each do |attr|
          neq(attr, pg_team, mongo_team, false)
        end


        %i(about achievement_count analytics benefit_description_1 benefit_description_2 benefit_description_3 benefit_name_1 benefit_name_2 benefit_name_3 big_image big_quote blog_feed branding country_id created_at endorsement_count facebook featured_banner_image featured_links_title github github_organization_name headline hide_from_featured highlight_tags hiring_tagline interview_steps invited_emails link_to_careers_page location monthly_subscription name number_of_jobs_to_show office_photos organization_way organization_way_name organization_way_photo our_challenge paid_job_posts premium preview_code reason_description_1 reason_description_2 reason_description_3 reason_name_1 reason_name_2 reason_name_3 size stack_list twitter upcoming_events upgraded_at valid_jobs website why_work_image your_impact youtube_url).each do |attr|
          neq(attr, pg_team, mongo_team)
        end
      rescue => ex

        ap ex

        require 'pry'; binding.pry
      end
    end
  end

  task counts: :environment do
    pg_team_count = PgTeam.count
    puts "PgTeam.count=#{pg_team_count}"
    team_count = Team.count
    puts "Team.count=#{team_count}"
    puts "Unmigrated teams count=#{(team_count - pg_team_count)}"
  end


  task unmigrated: :environment do
    unmigrated_teams = []

    Team.all.each do |team|
      unmigrated_teams << team.id.to_s unless PgTeam.where(mongo_id: team.id.to_s).exists?
    end

    puts "Unmigrated teams count=#{unmigrated_teams.count}"
    puts "Unmigrated Teams=%w(#{unmigrated_teams.join(' ')})"
  end
end
