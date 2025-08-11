{
  "reportingFramework": {
    "version": "2.0",
    "metadata": {
      "name": "Employee Reporting and Analytics Framework",
      "description": "Unified configuration for employee-centric reporting, supporting flexible personas, metrics, aggregation, and output formats."
    },
    "reusableComponents": {
      "employeePersonas": [
        {
          "id": "directs",
          "label": "Direct Reports",
          "description": "Employees reporting directly to a manager.",
          "criteria": {
            "managerLevel": 1
          }
        },
        {
          "id": "directs_and_below",
          "label": "Directs and Below",
          "description": "Employees reporting directly or indirectly to the user",
          "criteria": {
            "managerLevel": "all_levels_below"
          }
        },
        {
          "id": "all_employees",
          "label": "All Employees",
          "description": "All employees in the organization",
          "criteria": {
            "scope": "organization"
          }
        },
        {
          "id": "sales_leaders",
          "label": "Sales Leaders",
          "description": "Managers within the sales department.",
          "criteria": {
            "department": "Sales",
            "isManager": true
          }
        },
        {
          "id": "top_sales_performers",
          "label": "Top Sales Performers",
          "description": "Sales employees with A performance rating.",
          "criteria": {
            "department": "Sales",
            "performanceRating": "A"
          }
        },
        {
          "id": "new_hires_q4",
          "label": "New Hires (Q4)",
          "description": "Employees hired in Q4 2024.",
          "criteria": {
            "hireDate": {
              "operator": "BETWEEN",
              "startValue": "2024-10-01",
              "endValue": "2024-12-31"
            }
          }
        }
      ],
      "metrics": [
        {
          "id": "SALES_VOLUME",
          "name": "Sales Volume",
          "description": "Total value of deals closed.",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        },
        {
          "id": "SALES_CALLS",
          "name": "Number of Calls",
          "description": "Total number of sales calls made.",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        },
        {
          "id": "CLOSURE_RATE",
          "name": "Closure Rate",
          "description": "Average rate of successful deals.",
          "aggregationType": "AVG",
          "dataType": "PERCENTAGE"
        },
        {
          "id": "HEADCOUNT_START",
          "name": "Headcount Start",
          "description": "Employee count at the start of period.",
          "aggregationType": "COUNT",
          "dataType": "NUMBER"
        },
        {
          "id": "HEADCOUNT_END",
          "name": "Headcount End",
          "description": "Employee count at the end of period.",
          "aggregationType": "COUNT",
          "dataType": "NUMBER"
        },
        {
          "id": "VOLUNTARY_TERMINATIONS",
          "name": "Voluntary Leaves",
          "description": "Count of voluntary terminations.",
          "aggregationType": "COUNT",
          "dataType": "NUMBER"
        },
        {
          "id": "attendance",
          "name": "Attendance",
          "description": "Days present",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        },
        {
          "id": "hours_worked",
          "name": "Hours Worked",
          "description": "Total hours worked",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        }
      ],
      "postAggregationCalculations": [
        {
          "id": "SALES_PER_CALL_EFFICIENCY",
          "name": "Sales Per Call",
          "description": "Calculates the average sales value per call.",
          "formula": "SALES_VOLUME / SALES_CALLS",
          "outputDataType": "NUMBER"
        },
        {
          "id": "PERCENT_OF_TOTAL_SALES",
          "name": "% of Total Sales",
          "description": "Calculates a group's contribution to the total sales volume.",
          "formula": "(AGGREGATED_METRIC / Global.TOTAL_SALES_VOLUME) * 100",
          "outputDataType": "PERCENTAGE"
        },
        {
          "id": "RETENTION_RATE",
          "name": "Annual Retention Rate",
          "description": "Annual retention rate by department.",
          "formula": "(HEADCOUNT_END - VOLUNTARY_TERMINATIONS) / HEADCOUNT_START * 100",
          "outputDataType": "PERCENTAGE"
        },
        {
          "id": "BONUS_ELIGIBILITY",
          "name": "Bonus Eligibility",
          "description": "Eligibility for bonus based on attendance and sales.",
          "formula": "attendance > 20 && sales > 10000",
          "outputDataType": "BOOLEAN"
        },
        {
          "id": "AVG_GROUP_SALES",
          "name": "Average Group Sales",
          "description": "Average sales for group in period.",
          "formula": "sum(sales) / count(employee_ids)",
          "outputDataType": "NUMBER"
        }
      ]
    },
    "aggregation": {
      "periods": [
        "daily",
        "monthly",
        "quarterly",
        "annually",
        "custom"
      ],
      "levels": [
        "employee",
        "period",
        "group"
      ],
      "grouping": {
        "group_by": [
          "department",
          "location",
          "team"
        ]
      }
    },
    "reports": [
      {
        "id": "employee_overview",
        "name": "Employee Overview Report",
        "description": "Summary report of employees by persona",
        "filters": {
          "employeePersonas": ["directs", "all_employees"]
        },
        "columns": [
          { "name": "employeeId", "label": "Employee ID" },
          { "name": "name", "label": "Name" },
          { "name": "title", "label": "Title" },
          { "name": "department", "label": "Department" },
          { "name": "persona", "label": "Persona" }
        ],
        "scheduling": {
          "allowed": true,
          "defaultFrequency": "monthly"
        },
        "output": {
          "format": "json"
        }
      },
      {
        "id": "employee_retention",
        "name": "Employee Retention Analysis",
        "description": "Track retention metrics across departments",
        "period": {
          "type": "YEARLY",
          "startDate": "2024-01-01",
          "endDate": "2024-12-31"
        },
        "metrics": [
          "HEADCOUNT_START",
          "HEADCOUNT_END",
          "VOLUNTARY_TERMINATIONS"
        ],
        "aggregations": [
          {
            "groupName": "department",
            "groupBy": "departmentId",
            "metricsToReport": [
              "HEADCOUNT_START",
              "HEADCOUNT_END",
              "VOLUNTARY_TERMINATIONS"
            ]
          }
        ],
        "calculations": [
          {
            "calculationId": "RETENTION_RATE",
            "scope": "department"
          }
        ],
        "output": {
          "fields": [
            "departmentId",
            "departmentName",
            "HEADCOUNT_START",
            "HEADCOUNT_END",
            "VOLUNTARY_TERMINATIONS",
            "RETENTION_RATE"
          ],
          "format": "HTML"
        }
      },
      {
        "id": "q4_sales_performance",
        "name": "Q4 Sales Performance Report",
        "description": "Quarterly sales metrics for top performers",
        "period": {
          "type": "QUARTERLY",
          "startDate": "2024-10-01",
          "endDate": "2024-12-31"
        },
        "filters": {
          "employeePersona": "top_sales_performers"
        },
        "metrics": [
          "SALES_VOLUME",
          "SALES_CALLS",
          "CLOSURE_RATE"
        ],
        "aggregations": [
          {
            "groupName": "individual",
            "groupBy": "employeeId",
            "metricsToReport": [
              "SALES_VOLUME",
              "SALES_CALLS",
              "CLOSURE_RATE"
            ]
          },
          {
            "groupName": "team",
            "groupBy": "teamId",
            "metricsToReport": [
              "SALES_VOLUME",
              "CLOSURE_RATE"
            ]
          }
        ],
        "calculations": [
          {
            "calculationId": "SALES_PER_CALL_EFFICIENCY",
            "scope": "individual"
          }
        ],
        "output": {
          "fields": [
            "employeeId",
            "name",
            "team",
            "SALES_VOLUME",
            "SALES_CALLS",
            "CLOSURE_RATE",
            "SALES_PER_CALL_EFFICIENCY"
          ],
          "format": "CSV"
        }
      },
      {
        "id": "q4_sales_performance_efficiency",
        "name": "Q4 Sales Performance & Efficiency Report",
        "description": "Quarterly report for Q4 on sales performance and efficiency, including top performers and new hires.",
        "period": {
          "type": "QUARTERLY",
          "startDate": "2024-10-01",
          "endDate": "2024-12-31"
        },
        "personas": [
          "top_sales_performers",
          "new_hires_q4"
        ],
        "employeeFilter": {
          "type": "PERSONA",
          "personaId": "top_sales_performers"
        },
        "metrics": [
          "SALES_VOLUME",
          "SALES_CALLS",
          "CLOSURE_RATE"
        ],
        "aggregationGroups": [
          {
            "groupName": "IndividualAggregations",
            "groupBy": "employeeId",
            "metricsToReport": [
              "SALES_VOLUME",
              "SALES_CALLS",
              "CLOSURE_RATE"
            ]
          },
          {
            "groupName": "TeamAggregations",
            "groupBy": "teamId",
            "metricsToReport": [
              "SALES_VOLUME",
              "SALES_CALLS"
            ]
          },
          {
            "groupName": "DepartmentAggregations",
            "groupBy": "departmentId",
            "metricsToReport": [
              "SALES_VOLUME"
            ]
          }
        ],
        "postAggregationCalculations": [
          {
            "calculationId": "SALES_PER_CALL_EFFICIENCY",
            "scope": "IndividualAggregations"
          },
          {
            "calculationId": "PERCENT_OF_TOTAL_SALES",
            "scope": "DepartmentAggregations"
          }
        ],
        "output": {
          "fields": [
            "employeeId",
            "teamId",
            "departmentId",
            "SALES_VOLUME",
            "SALES_CALLS",
            "CLOSURE_RATE",
            "SALES_PER_CALL_EFFICIENCY",
            "PERCENT_OF_TOTAL_SALES"
          ],
          "format": "json"
        }
      }
    ]
  }
}


{
  "reportingFramework": {
    "version": "2.1",
    "metadata": {
      "name": "Enterprise Performance Analytics",
      "description": "Comprehensive reporting framework for employee and business metrics",
      "created": "2024-03-15",
      "lastUpdated": "2024-06-20"
    },
    
    "personas": [
      {
        "id": "top_performers",
        "name": "Top Sales Performers",
        "description": "Employees with highest performance ratings in sales",
        "filters": {
          "department": "Sales",
          "performanceRating": "A",
          "status": "active"
        }
      },
      {
        "id": "new_hires_q4",
        "name": "New Hires (Q4)",
        "description": "Employees hired during Q4 2024",
        "filters": {
          "hireDate": {
            "operator": "BETWEEN",
            "startValue": "2024-10-01",
            "endValue": "2024-12-31"
          }
        }
      },
      {
        "id": "engineering_leads",
        "name": "Engineering Leads",
        "description": "Technical team leads in engineering",
        "filters": {
          "department": "Engineering",
          "jobLevel": "Lead"
        }
      }
    ],
    
    "metrics": [
      {
        "id": "SALES_VOLUME",
        "name": "Total Sales Volume",
        "description": "Monetary value of closed deals",
        "dataType": "currency",
        "defaultAggregation": "SUM",
        "validations": {
          "minValue": 0
        }
      },
      {
        "id": "SALES_CALLS",
        "name": "Sales Calls",
        "description": "Number of sales calls made",
        "dataType": "integer",
        "defaultAggregation": "SUM",
        "validations": {
          "minValue": 0
        }
      },
      {
        "id": "CLOSURE_RATE",
        "name": "Deal Closure Rate",
        "description": "Percentage of successful deals",
        "dataType": "percentage",
        "defaultAggregation": "AVG",
        "validations": {
          "minValue": 0,
          "maxValue": 100
        }
      },
      {
        "id": "RETENTION_RATE",
        "name": "Employee Retention Rate",
        "description": "Percentage of employees retained",
        "dataType": "percentage",
        "defaultAggregation": "AVG"
      }
    ],
    
    "calculations": [
      {
        "id": "SALES_EFFICIENCY",
        "name": "Sales Efficiency",
        "description": "Revenue generated per sales call",
        "formula": "SALES_VOLUME / SALES_CALLS",
        "outputType": "currency",
        "scopes": ["individual", "team"]
      },
      {
        "id": "DEPARTMENT_CONTRIBUTION",
        "name": "Department Contribution",
        "description": "Percentage of total sales by department",
        "formula": "(department.SALES_VOLUME / global.SALES_VOLUME) * 100",
        "outputType": "percentage",
        "scopes": ["department"]
      }
    ],
    
    "reports": [
      {
        "id": "q4_sales_performance",
        "name": "Q4 Sales Performance Report",
        "description": "Quarterly sales metrics analysis",
        "category": "Sales",
        "period": {
          "type": "QUARTERLY",
          "startDate": "2024-10-01",
          "endDate": "2024-12-31"
        },
        "filters": {
          "persona": "top_performers",
          "additionalFilters": {
            "region": ["North", "East"]
          }
        },
        "metrics": [
          "SALES_VOLUME",
          "SALES_CALLS",
          "CLOSURE_RATE"
        ],
        "aggregations": [
          {
            "level": "individual",
            "groupBy": "employeeId",
            "metrics": ["SALES_VOLUME", "SALES_CALLS", "CLOSURE_RATE"]
          },
          {
            "level": "team",
            "groupBy": "teamId",
            "metrics": ["SALES_VOLUME", "CLOSURE_RATE"]
          },
          {
            "level": "department",
            "groupBy": "departmentId",
            "metrics": ["SALES_VOLUME"]
          }
        ],
        "calculations": [
          {
            "calculationId": "SALES_EFFICIENCY",
            "scope": "individual",
            "outputName": "SALES_PER_CALL"
          },
          {
            "calculationId": "DEPARTMENT_CONTRIBUTION",
            "scope": "department",
            "outputName": "SALES_CONTRIBUTION_PCT"
          }
        ],
        "output": {
          "format": "CSV",
          "fields": [
            "period",
            "employeeId",
            "employeeName",
            "team",
            "SALES_VOLUME",
            "SALES_CALLS",
            "CLOSURE_RATE",
            "SALES_PER_CALL"
          ],
          "scheduling": {
            "enabled": true,
            "frequency": "MONTHLY",
            "delivery": {
              "method": "email",
              "recipients": ["sales-managers@company.com"]
            }
          }
        }
      },
      {
        "id": "annual_retention",
        "name": "Annual Retention Report",
        "description": "Employee retention metrics by department",
        "category": "HR",
        "period": {
          "type": "YEARLY",
          "startDate": "2024-01-01",
          "endDate": "2024-12-31"
        },
        "metrics": [
          "HEADCOUNT_START",
          "HEADCOUNT_END",
          "VOLUNTARY_TERMINATIONS"
        ],
        "aggregations": [
          {
            "level": "department",
            "groupBy": "departmentId",
            "metrics": ["HEADCOUNT_START", "HEADCOUNT_END", "VOLUNTARY_TERMINATIONS"]
          }
        ],
        "calculations": [
          {
            "name": "retention_rate",
            "description": "Annual retention rate by department",
            "formula": "(HEADCOUNT_END - VOLUNTARY_TERMINATIONS) / HEADCOUNT_START * 100",
            "outputName": "RETENTION_RATE",
            "scope": "department"
          }
        ],
        "output": {
          "format": "HTML",
          "fields": [
            "departmentId",
            "departmentName",
            "HEADCOUNT_START",
            "HEADCOUNT_END",
            "VOLUNTARY_TERMINATIONS",
            "RETENTION_RATE"
          ],
          "visualizations": [
            {
              "type": "bar_chart",
              "title": "Department Retention Rates",
              "xAxis": "departmentName",
              "yAxis": "RETENTION_RATE"
            }
          ]
        }
      }
    ],
    
    "frameworkConfig": {
      "supportedPeriods": ["DAILY", "WEEKLY", "MONTHLY", "QUARTERLY", "YEARLY", "CUSTOM"],
      "aggregationLevels": ["individual", "team", "department", "organization"],
      "outputFormats": ["CSV", "JSON", "HTML", "PDF", "EXCEL"],
      "defaults": {
        "outputFormat": "CSV",
        "schedulingFrequency": "MONTHLY"
      }
    }
  }
}


{
  "reportingFramework": {
    "version": "1.0.0",
    "metadata": {
      "name": "Employee Metrics Reporting System",
      "description": "Defines the structure for all employee performance and metrics reports."
    },
    "reusableComponents": {
      "employeePersonas": [
        {
          "id": "directs",
          "label": "Direct Reports",
          "description": "Employees who report directly to a manager.",
          "criteria": {
            "managerLevel": 1,
            "status": "active"
          }
        },
        {
          "id": "sales_leaders",
          "label": "Sales Leaders",
          "description": "Managers within the sales department, typically responsible for team performance.",
          "criteria": {
            "department": "Sales",
            "isManager": true
          }
        },
        {
          "id": "q4_hires",
          "label": "Q4 New Hires",
          "description": "Employees hired in the last quarter of the year.",
          "criteria": {
            "hireDate": {
              "operator": "BETWEEN",
              "startValue": "2024-10-01",
              "endValue": "2024-12-31"
            }
          }
        }
      ],
      "metrics": [
        {
          "id": "SALES_VOLUME",
          "name": "Sales Volume",
          "description": "Total value of deals closed by an employee.",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        },
        {
          "id": "SALES_CALLS",
          "name": "Sales Calls",
          "description": "Total number of sales calls made.",
          "aggregationType": "SUM",
          "dataType": "NUMBER"
        },
        {
          "id": "CLOSURE_RATE",
          "name": "Closure Rate",
          "description": "The average rate of successful deals.",
          "aggregationType": "AVG",
          "dataType": "PERCENTAGE"
        }
      ],
      "postAggregationCalculations": [
        {
          "id": "SALES_PER_CALL_EFFICIENCY",
          "name": "Sales Per Call",
          "description": "Calculates the average sales value per call.",
          "formula": "SALES_VOLUME / SALES_CALLS",
          "outputMetricName": "SALES_PER_CALL_EFFICIENCY",
          "outputDataType": "NUMBER"
        },
        {
          "id": "PERCENT_OF_TOTAL_SALES",
          "name": "% of Total Sales",
          "description": "Calculates a group's contribution to the overall sales volume.",
          "formula": "(SALES_VOLUME / Global.SALES_VOLUME) * 100",
          "outputMetricName": "PERCENT_OF_TOTAL_SALES",
          "outputDataType": "PERCENTAGE"
        }
      ]
    },
    "reports": [
      {
        "id": "q4_sales_report_top_performers",
        "name": "Q4 Sales Performance & Efficiency Report",
        "description": "Detailed report on sales volume, calls, and efficiency for the top sales performers.",
        "filters": {
          "employee": {
            "type": "PERSONA",
            "personaId": "directs"
          },
          "date": {
            "type": "FIXED_RANGE",
            "startDate": "2024-10-01",
            "endDate": "2024-12-31"
          },
          "metricsToInclude": ["SALES_VOLUME", "SALES_CALLS", "CLOSURE_RATE"]
        },
        "aggregation": {
          "period": "MONTHLY",
          "levels": [
            {
              "groupingKey": "employeeId",
              "groupingName": "Individual Metrics",
              "metrics": ["SALES_VOLUME", "SALES_CALLS", "CLOSURE_RATE"]
            },
            {
              "groupingKey": "teamId",
              "groupingName": "Team Totals",
              "metrics": ["SALES_VOLUME", "SALES_CALLS"]
            }
          ]
        },
        "calculations": [
          {
            "calculationId": "SALES_PER_CALL_EFFICIENCY",
            "description": "Apply the reusable sales per call calculation.",
            "scope": "Individual Metrics",
            "metricOperands": {
              "SALES_VOLUME": "SALES_VOLUME",
              "SALES_CALLS": "SALES_CALLS"
            }
          }
        ],
        "output": {
          "columns": [
            { "key": "employeeId", "label": "Employee ID" },
            { "key": "employeeName", "label": "Employee Name" },
            { "key": "period", "label": "Period" },
            { "key": "SALES_VOLUME", "label": "Sales Volume" },
            { "key": "SALES_CALLS", "label": "Sales Calls" },
            { "key": "SALES_PER_CALL_EFFICIENCY", "label": "Sales per Call" }
          ],
          "format": "json"
        },
        "scheduling": {
          "enabled": true,
          "frequency": "MONTHLY",
          "dayOfMonth": 5
        }
      }
    ]
  }
}