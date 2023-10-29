---
title: "Migrating from Google Workspace and Outlook Mail to iCloud+"
date: 2023-10-19T09:13:09+01:00
draft: true
tags: ["Google Workspace", "Outlook", "Email", "Gmail", "iCloud", "imapsync"]
showToc: true
---

# Why I'm Migrating from Google Workspace and Outlook to iCloud+

## Google Workspace

My Google Workspace subscription is becoming increasingly more expensive 💰. The current price is £8.28 for the Business Standard plan, and next month this will be increasing to £9.20. This has become too expensive for me, and I've been looking for a cheaper alternative.

## Outlook Mail

I have an Outlook account which has been lying dormant for a long time now.

> Come on, don't judge me! We all have that one email account that we don't use anymore... right?! 😅 🥵

Anyways... This account has lots of old emails that I'd like to keep for archival purposes. Whilst migrating my Google Workspace account to iCloud+, I'd like to migrate these old emails too.

## iCloud+

I'm already in the Apple ecosystem, and I recently purchased iCloud+ for £0.99 per month. This provides 50GB of storage which is more than enough for my needs. I also noticed you can use your [own domain for email](https://support.apple.com/en-gb/HT212514) (which was the main reason I was using Google Workspace), and you can add up to 5 custom domains with 3 email addresses each! So I've decided to migrate my emails to iCloud+ to save money and simplify my setup.

I'd like to begin by thanking [Myles Gray](https://blah.cloud/) for his excellent [blog post](https://blah.cloud/miscellaneous/migrating-google-workspaces-to-icloud-custom-domain/) on his experience migrating from Google Workspace to iCloud+. I used this as a guide for my migration.

# Migrating Emails

Similar to Myles, I used [`imapsync`](https://imapsync.lamiral.info/) to migrate my emails to iCloud+. `imapsync` is a well established and battle tested tool for migrating emails using the IMAP protocol. Being actively maintained, free and [open-source](https://github.com/imapsync/imapsync), with lots of [documentation and examples](https://imapsync.lamiral.info/FAQ.d/), it seemed like the perfect tool for the job. I used the `imapsync` [container image](https://hub.docker.com/r/gilleslamiral/imapsync/) to migrate my emails. I'm now sponsoring the maintainer on GitHub too! :slight_smile:

Before getting started you **must** have done the following:

- Created an [iCloud Mail email address](https://support.apple.com/en-gb/guide/icloud/mmdd8d1c5c/icloud) (`@icloud.com`).
- Completed steps 1 and 2 of adding your [custom email domain](https://www.icloud.com/icloudplus/customdomain) to iCloud+.
- Generated an [app password](https://support.apple.com/en-gb/HT204397) for your iCloud+ account.
- Generated an [app password](https://support.google.com/accounts/answer/185833?hl=en) for your Google Workspace account.
- Generated an [app password](https://support.microsoft.com/en-us/account-billing/using-app-passwords-with-apps-that-don-t-support-two-step-verification-5896ed9b-4263-e681-128a-a6f2979a7944) for your Outlook account.
- Enabled [IMAP](https://support.google.com/mail/answer/7126229?hl=en-GB#zippy=%2Cstep-check-that-imap-is-turned-on) access in the Gmail settings.

> The app specific passwords are needed if your account has two-factor authentication enabled (It's 2023 people! You should have this enabled! 😅)

## Migrating Google Workspace Emails to iCloud Mail

Initially, I used the command below to migrate my emails from Google Workspace to iCloud Mail:

```bash
docker run gilleslamiral/imapsync imapsync --errorsmax 200 --user1 <Gmail Address> --password1 "<Gmail App Password>" --host2 imap.mail.me.com --ssl2 --user2 <iCloud Address> --password2 "<iCloud+ App Password>" --gmail1
```

However, once the sync finished I noticed that some of the emails were not in the right folders :confused: What I didn't realise was that as well as my custom labels (which are just IMAP folders), Gmail also applies an `INBOX` label to all emails 🙄. The default behaviour of `imapsync` is:

> The first label synced by imapsync goes to its corresponding folder but other labels are then ignored. By default imapsync syncs folders (Gmail labels) using the classical alphanumeric order.
>
> <cite>`imapsync` Gmail FAQ [^1]</cite>

This is why some of my emails were not in the right folders!

Since I now had emails in the wrong folders, I had to delete them from iCloud Mail and start again 🫠:

```bash
docker run gilleslamiral/imapsync imapsync --errorsmax 200 --user1 <iCloud Address> --password1 "<iCloud+ App Password>" --host1 imap.mail.me.com -ssl1 --host2 imap.mail.me.com --ssl2 --user2 <iCloud Address> --password2 "<iCloud+ App Password>" --delete1 --noexpungeaftereach
```

Now, I ran the sync again but this time using the `--folderlast` option to sync the `INBOX` label last:

```bash
docker run gilleslamiral/imapsync imapsync --errorsmax 200 --user1 <Gmail Address> --password1 "<Gmail App Password>" --host2 imap.mail.me.com --ssl2 --user2 <iCloud Address> --password2 "<iCloud+ App Password>" --gmail1 --folderlast "INBOX" --folderlast "[Gmail]/All Mail"
```

In the end, I synced 1109 emails (390.7 MiB) which took 23 minutes and 51 seconds. This was the amount once `imapsync` skipped cross duplicates (emails labelled multiple times). I relied heavily on the `--dry` option to test the sync before running it for real.

## Migrating Outlook Emails to iCloud Mail

Next, I migrated my Outlook emails to iCloud Mail. As the emails in this account were very old, I wanted to put them all into a single folder called `Other/Outlook|Personal`. I used the following command to do this:

```bash
docker run gilleslamiral/imapsync imapsync --errorsmax 200 --user1 <Outlook Address> --office1 --password1 "<Outlook App Password>" --host2 imap.mail.me.com --ssl2 --user2 <iCloud Address> --password2 "<iCloud+ App Password>" --regextrans2 "s/(.*)/Other\/Outlook|Personal/" --folder "Inbox"
```

This synced 422 emails which took only a couple of minutes to complete. This sync went a lot smoother than the last one! 😆 😅

## Correcting DNS Records

Now that I'd migrated my emails, I needed to update my DNS MX and TXT records to point to [iCloud Mail](https://support.apple.com/en-gb/102374). Returning to [iCloud+](https://www.icloud.com/icloudplus/) and on the Custom Email Domain setup, I completed step 3. I use Cloudflare for DNS and I was pleasantly surprised to find Apple provides integration with Cloudflare to automatically amend my DNS records for me! :smiley: I just had to confirm the changes and wait for the changes to propagate.

## Conclusion

In the end, I'd successfully migrated 1531 emails from Google Workspace and Outlook to iCloud+, and I'm saving money! 💸 🎉

# References

[^1]: https://imapsync.lamiral.info/FAQ.d/FAQ.Gmail.txt
