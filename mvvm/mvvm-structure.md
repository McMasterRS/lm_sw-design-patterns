---
layout: default
title: Structuring a Next.js SPA for MVVM
parent: Model-View-ViewModel
nav_order: 2
---

# Structuring a Next.js SPA for MVVM 

Given that Next.js is a React-based framework, it lends itself very well to the MVVM design pattern. In this section, we will explain how to structure a Next.js single-page application (SPA) to conform the MVVM architecture patterns.  

A default Next.js application has the following structure:  

```
-- mvvm-demo
  -- node_modules
  -- app
  -- public
```

We recommend modifying the default Next.js project structure when making use of the MVVM design pattern as shown below:  

```
-- mvvm-demo
  -- node_modules
  -- components
  -- models
  -- app
  -- viewmodels
  -- api
  -- utils
  -- config
  -- public
    -- assets
  -- styles
```

In this updated project structure:  

- The `models` directory contains the Models used in your project.
- The `app` directory and its subdirectories contain the Views.
- The `viewmodels` directory contains the ViewModel files associated with each View.
- The `components` directory contains customized or reusable React components.
- The `utils` directory contains common utility functions (e.g., handler functions for your custom components)
- The `config` directory contains constant data such as database details or the configuration settings needed for single sign-on, theme configuration, etc.
- The `styles` directory contains the CSS styles used in your application. We recommend moving the CSS style sheets from the `app` directory to this new `styles` directory.
- The `api` directory contains the API routes used to fetch, add, delete, or update data in your application. 
- The `public/assets` directory contains the assets used in your application including the favicon (that should be moved from the `app` directory) and any other images.
