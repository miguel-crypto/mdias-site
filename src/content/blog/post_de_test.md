---
author: FjellOverflow
pubDatetime: 2024-07-25T11:11:53Z
modDatetime: 2024-09-25T12:07:53Z
title: Make the most out of your MX Master 3S on Mac!
slug: make-the-most-out-of-your-mx-master-3s-on-mac
featured: true
draft: false
tags:
  - astro
  - blog
  - docs
description: Methods to improve the way you use your Logitech MX Master 3S mouse.
---

![Alt text](/src/assets/images/DSC08604.jpg "My MX Master 3S")

Logitech's MX Master line of productivity mice has been the _defacto_ choice for a lot professionals whose job heavily relies on a versatility when interacting with a computer, but are they using it to its full potential?

## The itch

I confess I have never cared much for computer mice for my job. I did at one point care a lot about gaming mice at the peak of my Counter-Strike days, and I had all sorts of trypophobia inducing, hole-filled ultra-lightweight gaming mice from experimental Chinese companies.

For non-gaming occasions I always felt that the fantastic MacBook trackpad sufficed, and it did. I've always used my MacBook as my keyboard + mouse combo, even at my desk setup at home, where I used to place my MacBook under my big monitor to have it centrally placed.

That changed.

At Cloudflare, I'm part of a team of seven Solutions Engineers. Out of these seven, five used a Logitech MX Master 3S. I often noticed this but didn't really understand the point of carrying around a heavy, big, somewhat clunky-looking mouse to work when all of us had access to a perfectly engineered trackpad.

## Setting up _Giscus_

_Giscus_ can be set up easily on [giscus.app](https://giscus.app/), but I will outline the process shortly still.

### Prequisites

Prequisites to get _Giscus_ working are

- the repository is [public](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility#making-a-repository-public)
- the [Giscus app](https://github.com/apps/giscus) is installed
- the [Discussions](https://docs.github.com/en/github/administering-a-repository/managing-repository-settings/enabling-or-disabling-github-discussions-for-a-repository) feature is turned on for your repository

If any of these conditions cannot be fulfilled for any reason, unfortunately, _Giscus_ cannot be integrated.

### Configuring _Giscus_

Next, configuring _Giscus_ is necessary. In most cases, the preselected defaults are suitable, and you should only modify them if you have a specific reason and know what you are doing. Don't worry too much about making the wrong choices; you can always adjust the configuration later on.

However you need to

- select the right language for the UI
- specify the _GitHub_ repository you want to connect, typically the repository containing your statically hosted _AstroPaper_ blog on _GitHub Pages_
- create and set an `Announcement` type discussion on _GitHub_ if you want to ensure nobody can create random comments directly on _GitHub_
- define the color scheme

After configuring the settings, _Giscus_ provides you with a generated `<script>` tag, which you will need in the next steps.

## Simple script tag

You should now have a script tag that looks like this:

```html
<script
  src="https://giscus.app/client.js"
  data-repo="[ENTER REPO HERE]"
  data-repo-id="[ENTER REPO ID HERE]"
  data-category="[ENTER CATEGORY NAME HERE]"
  data-category-id="[ENTER CATEGORY ID HERE]"
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="preferred_color_scheme"
  data-lang="en"
  crossorigin="anonymous"
  async
></script>
```

Simply add that to the source code of the site. Most likely, if you're using _AstroPaper_ and want to enable comments on posts, navigate to `src/layouts/PostDetails.astro` and paste it into the desired location where you want the comments to appear, perhaps underneath the `Share this post on:` buttons.

```diff
      <ShareLinks />
    </div>

+    <script src="https://giscus.app/client.js"
+        data-repo="[ENTER REPO HERE]"
+        data-repo-id="[ENTER REPO ID HERE]"
+        data-category="[ENTER CATEGORY NAME HERE]"
+        data-category-id="[ENTER CATEGORY ID HERE]"
+        ...
+    </script>

  </main>
  <Footer />
</Layout>
```

And it's done! You have successfully integrated comments in _AstroPaper_!

## React component with light/dark theme

The embedded script tag in the layout is quite static, with the _Giscus_ configuration, including `theme`, hardcoded into the layout. Given that _AstroPaper_ features a light/dark theme toggle, it would be nice for the comments to seamlessly transition between light and dark themes along with the rest of the site. To achieve this, a more sophisticated approach to embedding _Giscus_ is required.

Firstly, we are going to install the [React component](https://www.npmjs.com/package/@giscus/react) for _Giscus_:

```bash
npm i @giscus/react
```

Then we create a new `Comments.tsx` React component in `src/components`:

```tsx
import Giscus, { type Theme } from "@giscus/react";
import { GISCUS } from "@config";
import { useEffect, useState } from "react";

interface CommentsProps {
  lightTheme?: Theme;
  darkTheme?: Theme;
}

export default function Comments({
  lightTheme = "light",
  darkTheme = "dark",
}: CommentsProps) {
  const [theme, setTheme] = useState(() => {
    const currentTheme = localStorage.getItem("theme");
    const browserTheme = window.matchMedia("(prefers-color-scheme: dark)")
      .matches
      ? "dark"
      : "light";

    return currentTheme || browserTheme;
  });

  useEffect(() => {
    const mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
    const handleChange = ({ matches }: MediaQueryListEvent) => {
      setTheme(matches ? "dark" : "light");
    };

    mediaQuery.addEventListener("change", handleChange);

    return () => mediaQuery.removeEventListener("change", handleChange);
  }, []);

  useEffect(() => {
    const themeButton = document.querySelector("#theme-btn");
    const handleClick = () => {
      setTheme(prevTheme => (prevTheme === "dark" ? "light" : "dark"));
    };

    themeButton?.addEventListener("click", handleClick);

    return () => themeButton?.removeEventListener("click", handleClick);
  }, []);

  return (
    <div className="mt-8">
      <Giscus theme={theme === "light" ? lightTheme : darkTheme} {...GISCUS} />
    </div>
  );
}
```

This _React_ component not only wraps the native _Giscus_ component, but also introduces additional props, namely `lightTheme` and `darkTheme`. Leveraging two event listeners, the _Giscus_ comments will align with the site's theme, dynamically switching between dark and light themes whenever the site or browser theme is changed.

We also need to define the `GISCUS` config, for which the optimal location is in `src/config.ts`:

```ts
import type { GiscusProps } from "@giscus/react";

...

export const GISCUS: GiscusProps = {
  repo: "[ENTER REPO HERE]",
  repoId: "[ENTER REPO ID HERE]",
  category: "[ENTER CATEGORY NAME HERE]",
  categoryId: "[ENTER CATEGORY ID HERE]",
  mapping: "pathname",
  reactionsEnabled: "0",
  emitMetadata: "0",
  inputPosition: "bottom",
  lang: "en",
  loading: "lazy",
};
```

Note that specifying a `theme` here will override the `lightTheme` and `darkTheme` props, resulting in a static theme setting, similar to the previous approach of embedding _Giscus_ with the `<script>` tag.

To complete the process, add the new Comments component to `src/layouts/PostDetails.astro` (replacing the `script` tag from the previous step).

```diff
+ import Comments from "@components/Comments";

      <ShareLinks />
    </div>

+    <Comments client:only="react" />

  </main>
  <Footer />
</Layout>
```

And that's it!
