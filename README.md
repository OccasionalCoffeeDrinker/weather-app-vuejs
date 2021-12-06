# Front End Candidate Test Project

A simple weather app using [MetaWeather API](https://www.metaweather.com/api/).

## Goal

The goal of this project is to build a Weather application using the JavaScript Geolocation API and the MetaWeather API.

## Time

We hope you can spend about two hours on this project. If you can finish faster — great! If not, limit yourself and don't spend much longer than 2 hours MAX.

## Criteria

* Code readability
* Maintainability

## Requirements

- You can use any JS framework you want
- You can use an UI framework you want
- The application should work on Chrome, no need to work on browser compatibility
- The application does not have to be responsive
- You will have to use the MetaWeather API [https://www.metaweather.com/api/](https://www.metaweather.com/api/)
- Use the MetaWeather icons for the weather icons
- The provided design uses Bootstrap icons ([https://icons.getbootstrap.com/](https://icons.getbootstrap.com/)) for any other icons
- Bonus points for tests (unit, integration, ...)
- Bonus points for web accessibility
- Be prepared to explain your code and reasoning in a face-to-face interview
- Make sure your code installs and builds
- Add Readme instructions if needed

## Designs

We provided a Figma file for reference: https://www.figma.com/file/qiQhkqOaw1xqTBwxaBwzut/Frontend-interview-project?node-id=0%3A1. We want you to follow the designs as closely as possible.

## Structure

### client folder

This folder is empty, we want you to build your application inside this folder.

### server folder

As the MetaWeather API does not support CORS, we provided this server to help you make the API calls. To start the server, simply run `node server/index.js`.

To make API calls in your application, please use the following URL: `http://localhost:8080/https://www.metaweather.com/api`.

## Submit your solution

To submit your solution, please fork the project, push your code to your fork and create a Pull Request.

## Steps

### Step 1

Build a home page that will show today's weather for the current location

- Use the geolocation API to get the closest location around you
- Load today's weather
- Display the following values:
  - location
  - country
  - current temperature (it should be rounded up to the degree)
  - the weather icon (Please follow the `Icons` section in [https://www.metaweather.com/api/](https://www.metaweather.com/api/)).
  - Wind
  - Visibility
  - Humidity

### Step 2

Build a detail page that will show the weather for today and the next 5 days for the selected location

- Make the card clickable. When clicked, a new page should be loaded, displaying the weather for today and the next 5 days.
- This new page should show:
   - the day of the week
   - the weather icon
   - if the weather shows rain, snow or sleet, show the humidity level
   - the maximum and minimum temperature (they should be rounded up to the degree)
  
### Step 3 (Optional)

- Write tests.
- Make the application accessible to people with disabilities.
