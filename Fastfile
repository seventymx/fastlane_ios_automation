# You can find the documentation at https://docs.fastlane.tools
# https://docs.fastlane.tools/actions
# https://docs.fastlane.tools/plugins/available-plugins

default_platform(:ios)

# FASTLANE_DEBUG is set to true when it is not set at all
debug = ENV["FASTLANE_DEBUG"] != "false"
provisioning_name = debug ? "Development" : "AppStore"
output_dir = "../build/app/outputs/ipa"
cert_dir = "../../flutter_secrets/"
provisioning_profile_dir = "fastlane/profiles/"
provisioning_profile = "#{provisioning_name}_#{ENV["FASTLANE_APP_IDENTIFIER"]}.mobileprovision"
provisioning_profile_name = "#{ENV["FASTLANE_APP_IDENTIFIER"]} (#{provisioning_name})"

puts "Debug: #{debug}"
puts "Provisioning Name: #{provisioning_profile_name}"
puts "Provisioning Profile: #{provisioning_profile_dir}#{provisioning_profile}"

# The Dir function searches relative to Fastlane's default directory (ios/fastlane), but cert_dir is relative to ios directory - therefore we need to go up one directory
certificate = Dir["../#{cert_dir}#{provisioning_name}_*.cer"].first
certificate_id = nil

if (!certificate.nil?)
    puts "Certificate: fastlane/#{certificate}"

    certificate_id = File.basename(certificate, File.extname(certificate)).split("_")[1]
    puts "Certificate ID: #{certificate_id}"
end

platform :ios do
    desc "Update the Xcode project settings"
    lane :setup_project do
        if (!File.exist?("#{provisioning_profile_dir["fastlane/".length..-1]}#{provisioning_profile}"))
            UI.user_error!("No provisioning profile found. Please run the create_provisioning_profile lane first.")
        end

        # Print the environment variables
        puts "User: #{ENV["FASTLANE_USER"]}"
        puts "App Identifier: #{ENV["FASTLANE_APP_IDENTIFIER"]}"
        puts "ITC Team ID: #{ENV["FASTLANE_ITC_TEAM_ID"]}"
        puts "Team ID: #{ENV["FASTLANE_TEAM_ID"]}"

        # Ensure correct identifier is used
        update_app_identifier(xcodeproj: "Runner.xcodeproj", plist_path: "Runner/Info.plist", app_identifier: ENV["FASTLANE_APP_IDENTIFIER"])

        # Update display name
        update_info_plist(plist_path: "Runner/Info.plist", display_name: ENV["FASTLANE_APP_NAME"])

        # Update Project Team
        update_project_team(path: "Runner.xcodeproj", teamid: ENV["FASTLANE_TEAM_ID"])

        # Ensure correct provisioning profile is used
        update_project_provisioning(
            xcodeproj: "Runner.xcodeproj",
            profile: "#{provisioning_profile_dir}#{provisioning_profile}",
            build_configuration: debug ? "Debug" : "Release",
            code_signing_identity: debug ? "iPhone Developer" : "iPhone Distribution"
        )

        # Update code signing settings
        update_code_signing_settings(
            path: "Runner.xcodeproj",
            build_configurations: debug ? "Debug" : "Release",
            sdk: "iphoneos*",
            use_automatic_signing: false,
            code_sign_identity: debug ? "iPhone Developer" : "iPhone Distribution",
            profile_name: provisioning_profile_name,
            bundle_identifier: ENV["FASTLANE_APP_IDENTIFIER"]
        )
    end

    desc "Create and download the certificate"
    lane :create_certificate do
        # Return if the certificate already exists
        if (!certificate_id.nil?)
            puts "Certificate already exists"
            next
        end

        puts "Creating certificate..."

        # Ensure you have a valid certificate
        cert(username: ENV["FASTLANE_USER"], output_path: cert_dir, development: debug, keychain_password: ENV["FASTLANE_PASSWORD"])

        # Get the certificate ID
        relative_cert_dir = "../#{cert_dir}"
        certificate = Dir["#{relative_cert_dir}*.cer"].first
        certificate_id = File.basename(certificate, File.extname(certificate))

        # Renamed the certificate to include the provisioning name
        File.rename(certificate, "#{relative_cert_dir}#{provisioning_name}_#{certificate_id}.cer")
    end

    desc "Create and download the provisioning profile"
    lane :create_provisioning_profile do
        UI.user_error!("No certificate found. Please run the create_certificate lane first.") if (certificate_id.nil?)

        # Return if the provisioning profile already exists
        if (File.exist?("#{provisioning_profile_dir["fastlane/".length..-1]}#{provisioning_profile}"))
            puts "Provisioning Profile already exists"
            next
        end

        puts "Creating provisioning profile..."

        # Create App ID if it does not exist
        produce(username: ENV["FASTLANE_USER"], app_identifier: ENV["FASTLANE_APP_IDENTIFIER"], app_name: ENV["FASTLANE_APP_NAME"], skip_itc: true)

        # Download or create a provisioning profile
        sigh(
            app_identifier: ENV["FASTLANE_APP_IDENTIFIER"],
            provisioning_name: provisioning_profile_name,
            output_path: provisioning_profile_dir,
            username: ENV["FASTLANE_USER"],
            force: true,
            development: debug,
            team_id: ENV["FASTLANE_TEAM_ID"]
        )
    end
end
