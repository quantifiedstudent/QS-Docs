# Analyse Microservice
## Canvas Weather Graph Data
> Returns bundled daily weather data and student submissions form selected course in the specified time span by course id.
`http://localhost:{port}/graphCanvasWeather/course/:courseId?startDate={startDate}&endDate={endDate}` - API Route

>Example
`http://localhost:{port}/graphCanvasWeather/course/1234?startDate=2023-01-01&endDate=2023-10-10` - returns bundled daily weather data and student submissions form course with id 1234 for time period between 2023-01-01 and 2023-10-10
`http://localhost:{port}/graphCanvasWeather/course/1234?startDate=2023-01-01` - returns bundled daily weather data and student submissions form course with id 1234 for time period between 2023-01-01 and date of sending request

*startDate* - **Required**, specifies date of the first daily weather forecast
*endDate* - **Default is the date of sending request**, specifies date of the last daily weather forecast

| Parameter | Type | Description |
| ------------ | ------------ | ------------ |
| *courseId*  | int | Student Canvas Id (user_id) |
| *startDate* **Required** | string | Specifies date of the first daily weather forecast. |
| *endDate* | string | Specifies date of the last daily weather forecast. Default is the date of sending request. |