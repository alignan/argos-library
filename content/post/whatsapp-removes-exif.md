---
title: "Whatsapp Removes Exif properties"
date: 2017-11-01T22:45:08+01:00
draft: false
tags: [ "Whatsapp", "EXIF", "Tools", "Photos"]
---

# Whatsapp removes Exif properties

When backing up the photos from my mobile phone to my laptop, and then to `Google Photos` I noticed something odd: when syncing, most of the photos were shown as it were created just at that moment, even if some images were months older.

This is quite inconvenient.

Then I noticed not all the photos backed to `Google Photos` had this problem, and a deeper look at the [EXIF properties](https://en.wikipedia.org/wiki/Exif) of two different files confirmed my assumptions: [Whatsapp](https://www.whatsapp.com) removes these properties when sharing photos.

This is the `Exif properties` of a "good" photo:

[![](/img/whatsapp-removes-exif/00.png)](/img/whatsapp-removes-exif/00.png)

And a `Whatsapp` one:

[![](/img/whatsapp-removes-exif/01.png)](/img/whatsapp-removes-exif/01.png)

This has been discussed as well in [Quora](https://www.quora.com/Do-pictures-sent-on-WhatsApp-keep-their-EXIF-data).  Apparently this saves some extra bytes and help compressing the photos sent, and has some value in keeping privacy.  I understand the motivations, but it is **a Pain in the Ass**.

What properties am I talking about? simple ones, such as Creation and Modification date.

When syncing photos to `Google Photos`, if these values are not found, then the Cloud application just fills in the appropriate ones, using the sync time-stamp as new value, thus losing all time-relevancy when sorting photos.

The only option I have found is to install `Google Photos` in my mobile, and have `Whatsapp` photo folders to automatically sync, however for limited data budgets this may be an issue (not my issue currently).

My first approach was to try and automate something using [ExifTool](https://www.sno.phy.queensu.ca/~phil/exiftool/).  There is also a `Node.js` library called [exif-date-infer](https://www.npmjs.com/package/exif-date-infer), which seems able to infer the Exif date properties from the photos filename, and write back these properties to the file... but what is the file name convention used by `Whatsapp`? when importing the images to my laptop the images seem to be renamed as `IMG_XXXX`.
