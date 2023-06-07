---
title: "Simplifying Globalization: The Power of Standalone Localization Repositories"
date: 2021-02-21T16:14:59-04:00
draft: false
---
In today's globalized world, catering to a diverse audience is essential for businesses. To effectively serve localized content to users across the globe, it's crucial to streamline the localization process. This blog post explores a practical solution by pulling out localization files from your project and placing them in a standalone repository. We'll focus on using Rails project conventions for simplicity reasons.

## Motivation
Leading a significant feature in a shared code base often leads to the creation of a Long Standing Changelist (LSC). In such cases, translation files tend to be rapidly evolving due to other team members working on smaller fixes and features. This can become a major headache when merging changes with the upstream codebase.

To overcome this challenge and enable external vendors to access localization files for iterative improvements without touching our application's codebase, we have implemented a solution using **Git Submodules**. By referencing the localization files through submodules, we ensure seamless integration while allowing independent refinement of the files.

This approach helps us maintain a clean and efficient workflow, mitigating the complexities associated with merging changes and facilitating collaborative localization enhancements with external vendors.

## Streamlining Localization with Standalone Repositories

1. Create a Standalone Repository:
Begin by setting up a separate repository dedicated to localization files, mirroring the structure of your main codebase.
```
% i18n_locales repository
config/
├── README.md
├── locales
│   ├── de.yml
│   └── en.yml
...
```

2. Add the Localization Repository as a Submodule:
To integrate the standalone localization repository into your main codebase, add it as a submodule using the following command:
```
% git submodule add https://github.com/udaykadaboina/locale-factory.git lib/globalization
```

3. Update the Main Repository:
After adding the submodule, you'll notice the addition of a new file, `.gitmodules`, and a `lib/globalization` directory in your main repository.

4. Commit the Changes:
Commit the changes to your main repository, indicating the addition of the submodule:
```
git commit -m "Added locales as submodule"
```

5. Utilizing the Localization Repository:
Make the necessary changes in the standalone localization repository and use it as the main locale factory for your application.

6. Pulling Changes from Both Repositories:
To retrieve the latest code from both the main repository and the submodule repository simultaneously, use the following command:
```
git pull --recurse-submodules
```

7. Merging and Committing Changes:
Merge the changes from both repositories and commit your modifications along with the submodule updates:
```
git add lib/globalization
git commit -m "Syncing locale updates"
```

## Conclusion: Embrace the Flexibility
By separating your localization files from the main repository, you unlock greater flexibility for your team members to update these files with minimal merge conflicts. It also empowers external contributors to work independently, even outside your release cycle. Simplifying the globalization process ultimately leads to a more efficient and effective localized user experience.

## Resources
- [Git Submodules](https://git-scm.com/docs/git-submodule)
- [Rails i18n guide](https://guides.rubyonrails.org/i18n.html)
- [SpringBoot's Internationalization guide](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-internationalization.html)

