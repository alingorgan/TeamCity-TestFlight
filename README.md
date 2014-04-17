TeamCity-TestFlight
===================

## What's this?

This is a bash script which serves the purpose of automating the distribution process of iOS beta builds to beta testers or internal QA devices, through the well known beta distribution service, TestFlight.

## Purpose and usage

This script was designed to be used as an XCode Archive post action script in combination with a TeamCity server and local agent. Basically, once your archive has been built, you can use it to create a signed ipa file and upload it to TestFlight, under your account using the necessary tokens.

Using it is really simple. All you have to do is add it as a post-action to your archive build step, within your project scheme. Just make sure the project scheme is a duplicate of the release scheme.

For reference, see below:
![alt text](https://raw.github.com/alingorgan/TeamCity-TestFlight/master/sample_screenshot.png)

Also, for further reference, visit Matt Vlasach's [blog post](http://matt.vlasach.com/xcode-bots-hosted-git-repositories-and-automated-testflight-builds/) about CI and XCodeServer

#Credits

Kudos to Justin Miller and Matt Vlasach for making the first steps towards automating this process with XCode and XCode Server, which serve as a baseline for automating this process through TeamCity.

```bash

#!/bin/bash
#
# (Above line comes out when placing in Xcode scheme)
#
# Valid and working as of 04/17/2014
# Xcode 5.1, TeamCity Server via TeamCity agent
#
API_TOKEN="<Your TesFlight API Token>"
TEAM_TOKEN="<Your TestFlight Team Token>"
DISTRIBUTION_LISTS="<Comma separated TestFlight Distribution List Names for auto deploy>"
PROVISIONING_PROFILE="/Library/Server/Xcode/Data/ProvisioningProfiles/<your file name here>.mobileprovision"

SIGNING_IDENTITY="<your provisioning profile name here>"


# DO NOT EDIT BELOW HERE!
########################################

ARCHIVE_ORIGINAL_NAME="${ARCHIVE_PATH##*/}"

DSYM="/tmp/Archive.xcarchive/dSYMs/${PRODUCT_NAME}.app.dSYM"

IPA="/tmp/${PRODUCT_NAME}.ipa"

APP="/tmp/Archive.xcarchive/Products/Applications/${PRODUCT_NAME}.app"

# Clear out any old copies of the Archive
echo "Removing old Archive files from /tmp...";
/bin/rm -rf /tmp/Archive.xcarchive*


#Copy over the latest build the bot just created
echo "Looking for archive ${ARCHIVE_ORIGINAL_NAME} with path: ${ARCHIVE_PATH}"
echo "Copying latest archive ${ARCHIVE_ORIGINAL_NAME} to /tmp/${ARCHIVE_ORIGINAL_NAME}";
/bin/cp -Rp "${ARCHIVE_PATH}" "/tmp/"

#Renaming archive to generic Archive.xcarchive
/bin/mv "/tmp/${ARCHIVE_ORIGINAL_NAME}" "/tmp/Archive.xcarchive"

echo "Creating .ipa for ${PRODUCT_NAME}"
/bin/rm "${IPA}"
/usr/bin/xcrun -sdk iphoneos PackageApplication -v "${APP}" -o "${IPA}" --sign "${SIGNING_IDENTITY}" --embed "${PROVISIONING_PROFILE}"

echo "Done with IPA creation."

echo "Zipping .dSYM for ${PRODUCT_NAME}"
/bin/rm "${DSYM}.zip"
/usr/bin/zip -r "${DSYM}.zip" "${DSYM}"

echo "Created .dSYM for ${PRODUCT_NAME}"

echo "*** Uploading ${PRODUCT_NAME} to TestFlight ***"
/usr/bin/curl "http://testflightapp.com/api/builds.json" \
-F file=@"${IPA}" \
-F dsym=@"${DSYM}.zip" \
-F api_token="${API_TOKEN}" \
-F team_token="${TEAM_TOKEN}" \
-F distribution_lists="${DISTRIBUTION_LISTS}" \
-F notify=True \
-F notes="<Build changelog here>"

echo "TestFlight upload finished!"
```
