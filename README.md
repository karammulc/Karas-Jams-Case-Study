
<img src="https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Logos/Small-White-Logo.png" alt="Kara's Jams Logo" width="300">


## Project Overview
This project analyzes my personal Spotify listening data from May 2023 to June 2024. Through Python data consolidation, SQL transformations, and Looker Studio visualizations, I've created a comprehensive analysis of my music listening patterns, artist preferences, and temporal trends.

## Tools Used
- Python: Data consolidation
- BigQuery: SQL transformations
- Looker Studio: Visualization
- Google Sheets: Additional analysis

## Data Source
The data used for this project is my own historical Spotify data, obtained directly from Spotify account settings. The dataset contains just over a year of listening history between 5/31/23 to 6/01/24, with 38,489 observations before cleaning.

*Note: This analysis is based on personal Spotify listening data and represents individual usage patterns.*

## Data Processing Challenges and Solutions
The initial dataset presented several challenges requiring careful consideration. Working with multiple JSON files required a systematic approach to consolidation and cleaning. Using Python, I merged four separate JSON files into a single comprehensive dataset.

Time based analysis required careful handling of timestamps and creation of additional temporal categorizations. I implemented specific time of day classifications (Morning/Afternoon/Night) and extracted day of week information to enable deeper temporal analysis.

Data quality was maintained through comprehensive NULL value checks and consistent formatting across all columns. Zero duration plays were removed to ensure accurate listening time calculations.

## Data Transformation Process
### Initial Python Consolidation:
- Combined four JSON files into single dataset
- Converted data to CSV format
- Verified data integrity
- Created consistent column structure

### SQL Transformations:
Enhanced the data structure with additional calculated fields:
- Time of day categorization
- Day of week extraction
- Monthly aggregation features
- Custom date range parameters
- Listening duration calculations

### [Dashboard](https://public.tableau.com/app/profile/karam/viz/KarasJams/Dashboard1)
![Kara's Jams Dashboard page 1](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Final%20Dash%20P1.png)
![Kara's Jams Dashboard page 2](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Final%20Dash%20P2.png)

## Key Findings
Overall Statistics:
- Total listening time: 940 hours (39.18 days)
- Percent of time listening to music: 10.67%
- Total tracks played: 37,527

Top Artists:
1. Orion Sun (2,495 minutes)
2. Remi Wolf (1,358 minutes)
3. SZA (1,356 minutes)
4. Mt. Joy (1,285 minutes)
5. Doja Cat (1,216 minutes)

Listening Patterns:
- Peak listening time: Afternoons (50.5%)
- Most active day: Friday (17.2%)
- Night listening: 33.6%
- Morning listening: 16%

## Documentation
📝 [Detailed Cleaning & Exploration Process](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Cleaning%20%26%20Exploration.md)

## Dashboard Evolution

### Version 1
![Dashboard Version 1](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%202.png)

### Version 2
![Dashboard Version 2 p1](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P1.jpg)
![Dashboard Version 2 p2](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P1.jpg)
![Dashboard Version 2 p3](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P3.jpg)

### Final Dashboard
![Kara's Jams Dashboard page 1](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Final%20Dash%20P1.png)
![Kara's Jams Dashboard page 2](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Final_Dash%20P2.png)
