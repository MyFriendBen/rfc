# RFC 0005: Prepopulate data for 211 NC

Status: Draft
Author: Ming King
Created: 2025-09-22
PR: TODO

## Summary

We want the "More resources on 211’s website" link to pre-populate results with the user’s needs from the MyFriendBen questionnaire. This hybrid approach keeps 211’s resource logic and UI intact, while using MyFriendBen’s data to deliver more relevant results once users land on the 211 site.


## Background

In our partnership with NC 211, we identified a need to improve how users access short-term resources on the NC 211 co-brand. After collaborative discussions, CTD, MFB, and NC 211 agreed on an initial solution that makes short-term resources easier to access, while ensuring MFB’s site stays focused on its core goal of connecting users to long-term support without overwhelming them with too many additional resources.

- The current state of the system

    When users click the "For more local resources" link to visit the NC 211 website from our results page, they must reselect their needs and re-enter their location (e.g., ZIP code, address, or city) to view benefits.

- What we want to achieve

    - The link URL shall be dynamically constructed using the user's screener selections to pre-populate search results on NC 211's resource search page.

    - The query parameters shall include the user's zip code as the location parameter.

    - The query parameters shall include topics and subtopics based on the user's selected needs.

    - The application shall maintain a configuration mapping between MyFriendBen need types and 211's expected query parameters.

    - The application shall only pass non-personally identifiable information through URL parameters to 211's site.

    - The application shall gracefully handle cases where user data cannot be mapped to 211 parameters by redirecting to 211's general search page.

    - The application shall implement URL parameter length limits to prevent browser compatibility issues.

    - The application shall log link clicks for analytics without storing personal user data.

    ##### Note: The final goal is not included in this proposal.

## Proposal

Use the request data we’re sending to construct query parameters for the NC 211 website. Add functions to generate a valid search URL and update the \<a> tag in the `NcLink211Message` component. This ensures that when users click the "NC 211’s website" link, the redirected page receives the default needs query parameters.

```
<a href="https://nc211.org/" target="_blank" rel="noopener noreferrer">
    <FormattedMessage id="link211.clickHere" defaultMessage="NC 211's website." />
</a>
```

1. Extract the needs and ZIP code from the request data as follows:
    ```
    {
        "needs_food": true,
        "needs_baby_supplies": true,
        "needs_housing_help": false,
        "needs_mental_health_help": true,
        "needs_child_dev_help": false,
        "needs_funeral_help": false,
        "needs_family_planning_help": true,
        "needs_job_resources": false,
        "needs_dental_care": true,
        "needs_legal_services": false,
        "needs_veteran_services": false,
        "zipcode": 12345
    }
    ```

2.  Example 211 URL with query params: 

        "https://nc211.org/search/?keyword=healthcare&location=21754&searchBtn=Get+Help&distance=10&skip=0&subtopic=&topic=&taxonomyCode=notaxonomycodes"

        "https://nc211.org/search/?keyword=food&location=21754&distance=10&skip=0&subtopic=Food%2CHousing%2FShelter%2CMaterial%20Goods&topic=Basic%20Needs&taxonomyCode="

        "https://nc211.org/search/?keyword=food&location=21754&distance=10&skip=0&subtopic=Food%2CHousing%2FShelter%2CMaterial%20Goods%2CHuman%20Reproduction&topic=Basic%20Needs%2CHealth%20Care&taxonomyCode="

        "https://nc211.org/search/?keyword=food&location=21754&distance=10&skip=0&subtopic=Food%2CHousing%2FShelter%2CMaterial%20Goods%2CHuman%20Reproduction%2CLegal%20Services%2CMental%20Health%20Care%20Facilities%2CMental%20Health%20Support%20Services&topic=Basic%20Needs%2CCriminal%20Justice%20and%20Legal%20Services%2CEducation%2CHealth%20Care%2CIndividual%20and%20Family%20Life%2CMental%20Health%20and%20Substance%20Use%20Disorder%20Services&taxonomyCode="

    ##### Note: "%2C": join, "%20": space, "%2F": / 
    ##### Note: Note: Maximum URL length is not universal. Chrome supports up to 2,083 characters, whereas Firefox, Safari, and Opera can handle URLs over 60,000 characters.  

3.  Identify key components for building the final query URL:

    - base url: "https://nc211.org/search/"
    - keyword: "?keyword=topic"
    - location: "&location=zipcode&distance=10&skip=0" by default
    - search: "&searchBtn=Get+Help" (maybe optinal?)
    - subtopic: "&subtopic=Food%2CHousing%2FShelter%2CMaterial%20Goods" 
    - topic: "&topic="
    - ending url: "&taxonomyCode=" (can be empty?)

    ##### Note: If multiple topics are selected, only one will be passed as the keyword.

4. Mapped our needs to 211’s topics and subtopics in the following format: "Needs": "Topic" -> "Subtopic":

    #### Note: While some MFB needs align with 211’s topics and subtopics, the mappings below need further verification.
            
        "needs_food": "Basic Needs" -> "Food"
        "needs_baby_supplies": ? -> ?
        "needs_housing_help": "Basic Needs" -> "Housing/Shelter"
        "needs_mental_health_help": "Mental Health and Substance Use Disorder Services" -> ?
        "needs_child_dev_help": ? -> ?
        "needs_funeral_help": "Individual and Family Life" -> "Death Certification/Burial Arrangements"
        "needs_family_planning_help": ? -> ?
        "needs_job_resources": ? -> ?
        "needs_dental_care": ? -> ?
        "needs_legal_services": "Criminal Justice and Legal Services" -> "Legal Services"
        "needs_veteran_services": "Organizational/Community/International Services" -> "Military Service"


    <details>
    <summary> Expand to see 211’s keywords, topics, and subtopics </summary>

        [
            {
                "name": "Basic Needs",
                "taxonomyTerm": "B",
                "subtopics": [
                {
                    "name": "Food",
                    "taxonomyTerm": "BD",
                    "subtopics": null
                },
                {
                    "name": "Housing/Shelter",
                    "taxonomyTerm": "BH",
                    "subtopics": null
                },
                {
                    "name": "Material Goods",
                    "taxonomyTerm": "BM",
                    "subtopics": null
                },
                {
                    "name": "Transportation",
                    "taxonomyTerm": "BT",
                    "subtopics": null
                },
                {
                    "name": "Utilities",
                    "taxonomyTerm": "BV",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Consumer Services",
                "taxonomyTerm": "D",
                "subtopics": [
                {
                    "name": "Consumer Assistance and Protection",
                    "taxonomyTerm": "DD",
                    "subtopics": null
                },
                {
                    "name": "Consumer Regulation",
                    "taxonomyTerm": "DF",
                    "subtopics": null
                },
                {
                    "name": "Money Management",
                    "taxonomyTerm": "DM",
                    "subtopics": null
                },
                {
                    "name": "Tax Organizations and Services",
                    "taxonomyTerm": "DT",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Criminal Justice and Legal Services",
                "taxonomyTerm": "F",
                "subtopics": [
                {
                    "name": "Courts",
                    "taxonomyTerm": "FC",
                    "subtopics": null
                },
                {
                    "name": "Criminal Correctional System",
                    "taxonomyTerm": "FF",
                    "subtopics": null
                },
                {
                    "name": "Judicial Services",
                    "taxonomyTerm": "FJ",
                    "subtopics": null
                },
                {
                    "name": "Law Enforcement Agencies",
                    "taxonomyTerm": "FL",
                    "subtopics": null
                },
                {
                    "name": "Law Enforcement Services",
                    "taxonomyTerm": "FN",
                    "subtopics": null
                },
                {
                    "name": "Legal Assistance Modalities",
                    "taxonomyTerm": "FP",
                    "subtopics": null
                },
                {
                    "name": "Legal Expense Insurance",
                    "taxonomyTerm": "FS",
                    "subtopics": null
                },
                {
                    "name": "Legal Services",
                    "taxonomyTerm": "FT",
                    "subtopics": null
                },
                {
                    "name": "Legal Services Organizations",
                    "taxonomyTerm": "FV",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Education",
                "taxonomyTerm": "H",
                "subtopics": [
                {
                    "name": "Educational Institutions/Schools",
                    "taxonomyTerm": "HD",
                    "subtopics": null
                },
                {
                    "name": "Educational Programs",
                    "taxonomyTerm": "HH",
                    "subtopics": null
                },
                {
                    "name": "Educational Support Services",
                    "taxonomyTerm": "HL",
                    "subtopics": null
                },
                {
                    "name": "Postsecondary Instructional Programs",
                    "taxonomyTerm": "HP",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Environment and Public Health/Safety",
                "taxonomyTerm": "J",
                "subtopics": [
                {
                    "name": "Environmental Protection and Improvement",
                    "taxonomyTerm": "JD",
                    "subtopics": null
                },
                {
                    "name": "Public Health",
                    "taxonomyTerm": "JP",
                    "subtopics": null
                },
                {
                    "name": "Public Safety",
                    "taxonomyTerm": "JR",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Health Care",
                "taxonomyTerm": "L",
                "subtopics": [
                {
                    "name": "Emergency Medical Care",
                    "taxonomyTerm": "LD",
                    "subtopics": null
                },
                {
                    "name": "General Medical Care",
                    "taxonomyTerm": "LE",
                    "subtopics": null
                },
                {
                    "name": "Health Screening/Diagnostic Services",
                    "taxonomyTerm": "LF",
                    "subtopics": null
                },
                {
                    "name": "Health Supportive Services",
                    "taxonomyTerm": "LH",
                    "subtopics": null
                },
                {
                    "name": "Human Reproduction",
                    "taxonomyTerm": "LJ",
                    "subtopics": null
                },
                {
                    "name": "Inpatient Health Facilities",
                    "taxonomyTerm": "LL",
                    "subtopics": null
                },
                {
                    "name": "Medical Laboratories",
                    "taxonomyTerm": "LM",
                    "subtopics": null
                },
                {
                    "name": "Outpatient Health Facilities",
                    "taxonomyTerm": "LN",
                    "subtopics": null
                },
                {
                    "name": "Rehabilitation/Habilitation Services",
                    "taxonomyTerm": "LR",
                    "subtopics": null
                },
                {
                    "name": "Specialized Treatment and Prevention",
                    "taxonomyTerm": "LT",
                    "subtopics": null
                },
                {
                    "name": "Specialty Medicine",
                    "taxonomyTerm": "LV",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Income Support and Employment",
                "taxonomyTerm": "N",
                "subtopics": [
                {
                    "name": "Employment",
                    "taxonomyTerm": "ND",
                    "subtopics": null
                },
                {
                    "name": "Public Assistance Programs",
                    "taxonomyTerm": "NL",
                    "subtopics": null
                },
                {
                    "name": "Social Insurance Programs",
                    "taxonomyTerm": "NS",
                    "subtopics": null
                },
                {
                    "name": "Temporary Financial Assistance",
                    "taxonomyTerm": "NT",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Individual and Family Life",
                "taxonomyTerm": "P",
                "subtopics": [
                {
                    "name": "Death Certification/Burial Arrangements",
                    "taxonomyTerm": "PB",
                    "subtopics": null
                },
                {
                    "name": "Domestic Animal Services",
                    "taxonomyTerm": "PD",
                    "subtopics": null
                },
                {
                    "name": "Individual and Family Support Services",
                    "taxonomyTerm": "PH",
                    "subtopics": null
                },
                {
                    "name": "Leisure Activities/Recreation",
                    "taxonomyTerm": "PL",
                    "subtopics": null
                },
                {
                    "name": "Mutual Support",
                    "taxonomyTerm": "PN",
                    "subtopics": null
                },
                {
                    "name": "Social Development and Enrichment",
                    "taxonomyTerm": "PS",
                    "subtopics": null
                },
                {
                    "name": "Spiritual Enrichment",
                    "taxonomyTerm": "PV",
                    "subtopics": null
                },
                {
                    "name": "Volunteer Development",
                    "taxonomyTerm": "PW",
                    "subtopics": null
                },
                {
                    "name": "Volunteer Opportunities",
                    "taxonomyTerm": "PX",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Mental Health and Substance Use Disorder Services",
                "taxonomyTerm": "R",
                "subtopics": [
                {
                    "name": "Counseling Approaches",
                    "taxonomyTerm": "RD",
                    "subtopics": null
                },
                {
                    "name": "Counseling Settings",
                    "taxonomyTerm": "RF",
                    "subtopics": null
                },
                {
                    "name": "Mental Health Assessment and Treatment",
                    "taxonomyTerm": "RP",
                    "subtopics": null
                },
                {
                    "name": "Mental Health Care Facilities",
                    "taxonomyTerm": "RM",
                    "subtopics": null
                },
                {
                    "name": "Mental Health Support Services",
                    "taxonomyTerm": "RR",
                    "subtopics": null
                },
                {
                    "name": "Substance Use Disorder Services",
                    "taxonomyTerm": "RX",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Organizational/Community/International Services",
                "taxonomyTerm": "T",
                "subtopics": [
                {
                    "name": "Arts and Culture",
                    "taxonomyTerm": "TA",
                    "subtopics": null
                },
                {
                    "name": "Community Economic Development and Finance",
                    "taxonomyTerm": "TB",
                    "subtopics": null
                },
                {
                    "name": "Community Facilities/Centers",
                    "taxonomyTerm": "TC",
                    "subtopics": null
                },
                {
                    "name": "Community Groups and Government/Administrative Offices",
                    "taxonomyTerm": "TD",
                    "subtopics": null
                },
                {
                    "name": "Community Planning and Public Works",
                    "taxonomyTerm": "TE",
                    "subtopics": null
                },
                {
                    "name": "Community Recognition",
                    "taxonomyTerm": "TF",
                    "subtopics": null
                },
                {
                    "name": "Disaster Services",
                    "taxonomyTerm": "TH",
                    "subtopics": null
                },
                {
                    "name": "Donor Services",
                    "taxonomyTerm": "TI",
                    "subtopics": null
                },
                {
                    "name": "Information Services",
                    "taxonomyTerm": "TJ",
                    "subtopics": null
                },
                {
                    "name": "International Affairs",
                    "taxonomyTerm": "TL",
                    "subtopics": null
                },
                {
                    "name": "Military Service",
                    "taxonomyTerm": "TM",
                    "subtopics": null
                },
                {
                    "name": "Occupational/Professional Associations",
                    "taxonomyTerm": "TN",
                    "subtopics": null
                },
                {
                    "name": "Organizational Development and Management Delivery Methods",
                    "taxonomyTerm": "TO",
                    "subtopics": null
                },
                {
                    "name": "Organizational Development and Management Services",
                    "taxonomyTerm": "TP",
                    "subtopics": null
                },
                {
                    "name": "Political Organization and Participation",
                    "taxonomyTerm": "TQ",
                    "subtopics": null
                },
                {
                    "name": "Research",
                    "taxonomyTerm": "TR",
                    "subtopics": null
                }
                ]
            },
            {
                "name": "Target Populations",
                "taxonomyTerm": "Y",
                "subtopics": [
                {
                    "name": "Age Groups",
                    "taxonomyTerm": "YB",
                    "subtopics": null
                },
                {
                    "name": "Agencies/Organizations as Recipients",
                    "taxonomyTerm": "YA",
                    "subtopics": null
                },
                {
                    "name": "Benefits Recipients",
                    "taxonomyTerm": "YC",
                    "subtopics": null
                },
                {
                    "name": "Caregivers",
                    "taxonomyTerm": "YD",
                    "subtopics": null
                },
                {
                    "name": "Citizenship",
                    "taxonomyTerm": "YE",
                    "subtopics": null
                },
                {
                    "name": "Disabilities and Health Conditions",
                    "taxonomyTerm": "YF",
                    "subtopics": null
                },
                {
                    "name": "Educational Status",
                    "taxonomyTerm": "YG",
                    "subtopics": null
                },
                {
                    "name": "Ethnic Groups/National Origin",
                    "taxonomyTerm": "YH",
                    "subtopics": null
                },
                {
                    "name": "Experiencers of Paranormal/Extraterrestrial Events",
                    "taxonomyTerm": "YI",
                    "subtopics": null
                },
                {
                    "name": "Families and Individuals Needing Support",
                    "taxonomyTerm": "YJ",
                    "subtopics": null
                },
                {
                    "name": "Family Relationships",
                    "taxonomyTerm": "YK",
                    "subtopics": null
                },
                {
                    "name": "Income/Employment Status",
                    "taxonomyTerm": "YL",
                    "subtopics": null
                },
                {
                    "name": "Living Situation/Housing Status",
                    "taxonomyTerm": "YM",
                    "subtopics": null
                },
                {
                    "name": "Military Personnel/Contractors",
                    "taxonomyTerm": "YN",
                    "subtopics": null
                },
                {
                    "name": "Occupations",
                    "taxonomyTerm": "YO",
                    "subtopics": null
                },
                {
                    "name": "Offenders",
                    "taxonomyTerm": "YP",
                    "subtopics": null
                },
                {
                    "name": "Organizational/Practitioner Perspectives",
                    "taxonomyTerm": "YQ",
                    "subtopics": null
                },
                {
                    "name": "Religious Groups/Communities",
                    "taxonomyTerm": "YR",
                    "subtopics": null
                },
                {
                    "name": "Sex/Gender",
                    "taxonomyTerm": "YS",
                    "subtopics": null
                },
                {
                    "name": "Sexual Orientation/Gender Identity",
                    "taxonomyTerm": "YT",
                    "subtopics": null
                },
                {
                    "name": "Topical Identifiers/Issues",
                    "taxonomyTerm": "YZ",
                    "subtopics": null
                },
                {
                    "name": "Transients",
                    "taxonomyTerm": "YV",
                    "subtopics": null
                },
                {
                    "name": "Urban/Rural Location",
                    "taxonomyTerm": "YW",
                    "subtopics": null
                },
                {
                    "name": "Victims/Survivors",
                    "taxonomyTerm": "YX",
                    "subtopics": null
                },
                {
                    "name": "Volunteers",
                    "taxonomyTerm": "YY",
                    "subtopics": null
                }
                ]
            }
        ]

    </details>        

### Abandoned Ideas

#### Shared Storage (Local / Session)

At first, I considered using Django Channels with WebSockets to bypass URL length limits, leveraging a Producer and Consumer across tabs in the same session. However, I realized I cannot control the consumer side, the 211 code that opens the browser.

## Implementation

- Data structures

    Create a constant hash to map our needs to 211 NC’s topics and subtopics.

    ```
    const NEEDS_MAPPING = {
    needs_food: {
        topic: "Basic%20Needs",
        subtopic: "Food",
    },
    needs_housing_help: {
        topic: "Basic%20Needs",
        subtopic: "Housing%2FShelter",
    },
    needs_baby_supply: {
        topic: "Basic%20Needs",
        subtopic: "Baby%20Supplies",
    },
    funeral: {
        topic: "Individual%20and%20Family%20Life",
        subtopic: "Death%20Certification%2FBurial%20Arrangements",
    },
    ...
    };
    ```

    Save the mapped needs in a hash with topics as keys and subtopics as array values, like this:

    ```
    {
        "Basic%20Needs": ["Food", "Housing%2FShelter"],
        "Individual%20and%20Family%20Life": ["Death%20Certification%2FBurial%20Arrangements"]
        ...
    }
    ```

    Join the subtopics array into a string.

    ```
    {
        "Basic%20Needs": "Food%2CHousing%2FShelter",
        "Individual%20and%20Family%20Life": "Death%20Certification%2FBurial%20Arrangements"
    }
    ```

    Set variables and build the final URL to return.

    ```
    const BASE_URL = "https://nc211.org/search/?"
    const keyword = Object.keys(urls)[0];
    const location = "&location=${zipcode}&distance=10&skip=0"
    const topic = Object.keys(urls).join("%2C");
    const subtopic = Object.values(urls).join("%2C");
    const search = "&searchBtn=Get+Help"(Seems optional, need to test and confirm)
    const ending_url = "&taxonomyCode="

    const queryUrl = BASE_URL + keyword + search + location + topic + subtopic + ending_url

    ```

### Code Examples

```
    const NEEDS_MAPPING = {
        needs_food: {
            topic: "Basic%20Needs",
            subtopic: "Food",
        },
        needs_housing_help: {
            topic: "Basic%20Needs",
            subtopic: "Housing%2FShelter",
        },
        needs_baby_supply: {
            topic: "Basic%20Needs",
            subtopic: "Baby%20Supplies",
        },
        funeral: {
            topic: "Individual%20and%20Family%20Life",
            subtopic: "Death%20Certification%2FBurial%20Arrangements",
        },
        ...
    };

    const needsOptions = {
        "needs_food": true,
        "needs_housing_help": true,
        "needs_baby_supply": false,
        "funeral": true,
        ...
    }

    const householdNeeds = Object.entries(needsOptions)
        .filter(([key, value]) => value) // only truthy ones
        .map(([key]) => NEEDS_MAPPING[key]);


    const groupedNeeds = householdNeeds.reduce((acc, { topic, subtopic }) => {
        if (!acc[topic]) acc[topic] = [];
        acc[topic].push(subtopic);
        return acc;
    }, {});

    
    /**
     *    `groupedNeeds` returns a grouped needs hash like below, 
     *    {
     *        "Basic%20Needs": ["Food", "Housing%2FShelter"],
     *        "Individual%20and%20Family%20Life": ["Death%20Certification%2FBurial%20Arrangements"]
     *    }
     */


    const urlsHash = Object.fromEntries(
    Object.entries(groupedNeeds).map(([topic, subs]) => [
        topic,
        subs.join("%2C")
    ])
    );

    /**
     *    `urlsHash` looks like below,
     *    {
     *      "Basic%20Needs": "Food%2CHousing%2FShelter",
     *      "Individual%20and%20Family%20Life": "Death%20Certification%2FBurial%20Arrangements"
     *    }
     */

    const BASE_URL = "https://nc211.org/search/?"
    const ending_url = "&taxonomyCode="
    const keyword = Object.keys(urlsHash)[0];
    const location = "&location=${zipcode}&distance=10&skip=0"
    const search = "&searchBtn=Get+Help"
    const topic = Object.keys(urlsHash).join("%2C");
    const subtopic = Object.values(urlsHash).join("%2C");

    const queryUrl = BASE_URL + keyword + search + location + topic + subtopic + ending_url

    return (
        <div>
            <p>
                <FormattedMessage id="link211.message" defaultMessage="For more local resources please visit " />
                <a href={queryUrl} target="_blank" rel="noopener noreferrer">
                    <FormattedMessage id="link211.clickHere" defaultMessage="NC 211's website." />
                </a>
            </p>
        </div>
    );
```