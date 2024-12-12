+++
date = '2024-12-12T17:16:59Z'
draft = false
title = 'Lychee'
description = 'Link checking'
tags = ["blog", "lychee"]
+++

In the [Publishing]({{< relref "publishing" >}}) post I explain my [Github Actions](https://github.com/features/actions) publishing pipeline for this blog. I just added the awesome [Lychee](https://github.com/lycheeverse/lychee) to the pipeline for link checking. I've added a [job step](https://github.com/lycheeverse/lychee-action) to check all the links after Hugo builds the site and this will fail the pipeline if any of the checks fail. 

There are some link checks that will fail every time - for instance, Cloudflare links throw a {{< mark >}}403{{< /mark >}}. I'm guessing that this is because Cloudflare have banned GitHub hosted runner IP ranges due to abuse. To resolve this I've added a {{< mark >}}.lycheeignore{{< /mark >}} file to the root of the source repo. Because I'm checking out the source and build repositories into workspace subdirectories I've also added a step to copy this file to the workspace root.

Lychee emits a markdown table that GitHub actions automatically picks up and adds to the job summary page which is quite nice.

```markdown 
# Summary

| Status        | Count |
|---------------|-------|
| 🔍 Total      | 172   |
| ✅ Successful | 168   |
| ⏳ Timeouts   | 0     |
| 🔀 Redirected | 0     |
| 👻 Excluded   | 4     |
| ❓ Unknown    | 0     |
| 🚫 Errors     | 0     |
```

# Summary

| Status        | Count |
|---------------|-------|
| 🔍 Total      | 172   |
| ✅ Successful | 168   |
| ⏳ Timeouts   | 0     |
| 🔀 Redirected | 0     |
| 👻 Excluded   | 4     |
| ❓ Unknown    | 0     |
| 🚫 Errors     | 0     |