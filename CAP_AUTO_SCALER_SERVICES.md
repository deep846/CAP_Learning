watch these video --> 

                        https://www.youtube.com/watch?v=XFR-Gq_IC3I

                        https://www.youtube.com/watch?v=9TOghdThgJg&t=12s





					Auto Scaler Services.
					
					0. Create a service for autoscalling
					1. Create a policy.json file and paste the below content
					2. there might me 2 types of scalling
								1. ---> Dynamic Scaling (it's also 2 types)
										i> Metric Type memory consumed
												{
													"instance_min_count": 1, 
													"instance_max_count": 5, 
													"scaling_rules": [ 
														{
															"metric_type": "memoryused",
															"threshold": 90,
															"operator": ">=",
															"adjustment": "+1"
														},
												 {
															"metric_type": "memoryused", 
															"threshold": 30, 
															"operator": "<", 
															"adjustment": "-1" 
														}
													  
													]
												}
										ii> Metric Type memory utilized
											
												{
													"instance_min_count": 1, 
													"instance_max_count": 5, 
													"scaling_rules": [ 
													   {
															"metric_type": "memoryutil",
															"threshold": 90,
															"operator": ">=",
															"adjustment": "+1"
														},
												  {
															"metric_type": "memoryutil", 
															"threshold": 30, 
															"operator": "<", 
															"adjustment": "-1" 
														}
													   
													]
												}
										iii> Metric Type throughput
												{
													"instance_min_count": 1, 
													"instance_max_count": 5, 
													"scaling_rules": [ 
												  
													   {
															"metric_type": "throughput",
															"threshold": 100, // the throughput value in Requests/second
															"operator": ">",
															"adjustment": "+1"
														},
														{
															"metric_type": "throughput",
															"threshold": 100,
															"operator": "<=",
															"adjustment": "-1"
														}
													]
												}
										iv> Metric Type responsetime
												{
													"instance_min_count": 1, 
													"instance_max_count": 5, 
													"scaling_rules": [ 
													   {
															"metric_type": "responsetime",
															"threshold": 900, // the response time value in milliseconds
															"operator": ">",
															"adjustment": "+1"
														},
														{
															"metric_type": "responsetime",
															"threshold": 900,
															"operator": "<=",
															"adjustment": "-1"
														}
													  ]
												 }
										v> Advanced
										
												 {
													"instance_min_count": 1, 
													"instance_max_count": 5, 
													"scaling_rules": [ 
														{
															"metric_type": "memoryused",
															"breach_duration_secs": 600,
															"threshold": 90,
															"operator": ">=",
															"cool_down_secs": 300,
															"adjustment": "+1"
														},
												 {
															"metric_type": "memoryused", 
															"breach_duration_secs": 600, 
															"threshold": 30, 
															"operator": "<", 
															"cool_down_secs": 300, 
															"adjustment": "-1" 
														}
													   
													]
												}

												
								2. ---> Schedule-Based Scaling Policy
								
												{
												 "instance_min_count": 1, 
												 "instance_max_count": 5, 
												   "schedules": {  
												   "timezone":"Asia/Shanghai",
												   "recurring_schedule":[  
													  {  
														 "start_time":"10:00",
														 "end_time":"18:00",
														 "days_of_week":[  
															1,
															2,
															3
														 ],
														 "instance_min_count":1,
														 "instance_max_count":10,
														 "initial_min_instance_count":5
													  }      
													 ]
												   }
												}

									
									/////////////////////////////////Schedule with Start Date and Day of the Month///////////////////////////////////
									
															"schedules": {  
																   "timezone":"Asia/Shanghai",
																   "recurring_schedule":[  
																	   {  
																		 "start_date":"2016-06-27",
																		 "end_date":"2016-07-23",
																		 "start_time":"11:00",
																		 "end_time":"19:30",
																		 "days_of_month":[  
																			5,
																			15,
																			25
																		 ],
																		 "instance_min_count":3,
																		 "instance_max_count":10,
																		 "initial_min_instance_count":5
																	  }]}
																	  
																	  
									////////////////////////////////Specific Date Schedule
															{
																"instance_min_count": 1, 
																"instance_max_count": 5, 
															 "schedules":{  
															   "timezone":"Asia/Shanghai",
															"specific_date":[
																	  {
																		 "start_date_time":"2015-06-02T10:00", 
																		 "end_date_time":"2015-06-15T13:59", 
																		 "instance_min_count":1,
																		 "instance_max_count":4,
																		 "initial_min_instance_count":2
																	  },
																	  {
																		 "start_date_time":"2015-01-04T20:00",
																		 "end_date_time":"2015-02-19T23:15",
																		 "instance_min_count":2,
																		 "instance_max_count":5,
																		 "initial_min_instance_count":3
																	  }
																]
															  }
															}
															
															
									/////////////////////////////////Advanced Usage
									
									
																{
																 "instance_min_count": 1, 
																 "instance_max_count": 5, 
																   "schedules": {  
																   "timezone":"Asia/Shanghai",
																   "recurring_schedule":[  
																	  {  
																		 "start_time":"10:00",
																		 "end_time":"18:00",
																		 "days_of_week":[  
																			1,
																			2,
																			3
																		 ],
																		 "instance_min_count":1,
																		 "instance_max_count":10,
																		 "initial_min_instance_count":5
																	  },
																	  {  
																		 "start_date":"2016-06-27",
																		 "end_date":"2016-07-23",
																		 "start_time":"11:00",
																		 "end_time":"19:30",
																		 "days_of_month":[  
																			5,
																			15,
																			25
																		 ],
																		 "instance_min_count":3,
																		 "instance_max_count":10,
																		 "initial_min_instance_count":5
																	   }
																	 ]
																   }
																}

								3. now add these settings in mta yaml
									
									first it should be under resurece
											
											      - name: externalServicesConnections-autoscaler
													parameters:
													  path: './policy.json <path of policy .json file>'
													  
													  
													then in the below we need to define our service like others
													
														  - name: externalServicesConnections-autoscaler
															type: org.cloudfoundry.managed-service
															parameters:
															  service: autoscaler
															  service-plan: standard
								4. now do : 
										mbt build
										cf deploy mta_archive/filename.mtar