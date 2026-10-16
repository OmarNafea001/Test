2026-10-16 16:01:21,233 [JobManager.Wrk.55504] WARN  wedo.ecc.im.services.core.LoaderTask.[Loader] IM_LD0007 [] - Loader execution failed: Error during remote execution -> Failed to execute BPM flow 'LD_Digital_Journeys_TIBCO': Missing 'field' attribute in GreaterThan rule
2026-10-16 16:01:21,266 [JobManager.Wrk.55504] ERROR wedo.bpm.services.flowstore.FlowStoreBase.[BPM.FlowStore] BPM_FS0204 [] - Task (run id=265,629,969) for flow (run id=121,156) ended on error.
2026-10-16 16:01:21,269 [JobManager.Wrk.55504] ERROR wedo.bpm.services.flowstore.FlowStoreBase.[BPM.FlowStore] BPM_FS0109 [] - Ending flow (run id=121,156) on error.
2026-10-16 16:01:21,269 [JobManager.Wrk.55504] ERROR wedo.bpm.services.flow.FlowManager.[BPM.FlowMngr] JAF_G1000 [] -


the below error is appearing when i use the following xml : 

"<?xml version="1.0"?>
<flow name="LD_Digital_Journeys_TIBCO" version="1">
	<flowdescription valueof="LD_Digital_Journeys_TIBCO"></flowdescription>
	<engine class="wedo.bpm.services.engine.StandardProcess"></engine>
	<configuration>
		<AutoRefresh value="1"></AutoRefresh>
		<RefreshDelay value="30"></RefreshDelay>
		<ShowErrorMessages value="1"></ShowErrorMessages>
		<ShowInformationMessages value="1"></ShowInformationMessages>
		<ShowRunWindow value="1"></ShowRunWindow>
		<DefaultEntryName value=""></DefaultEntryName>
		<SecurityPermissions>
			<Visible></Visible>
			<Modify></Modify>
			<Execute></Execute>
		</SecurityPermissions>
	</configuration>
	<flowsharedarea>
		<parameter name="ALARM_MODULE" type="String">RAID</parameter>
		<parameter name="ALARM_CONTEXT_TYPE" type="String">ALARMS</parameter>
	</flowsharedarea>
	<aim>
		<parameter name="Name" type="String"></parameter>
		<parameter name="Description" type="String"></parameter>
		<parameter name="Elements" type="Array">
			<parameter name="1" type="Map">
				<parameter name="Name" type="String">rdr_correlationId</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="2" type="Map">
				<parameter name="Name" type="String">rdr_MSISDN</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="3" type="Map">
				<parameter name="Name" type="String">rdr_TIMESTAMP</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="4" type="Map">
				<parameter name="Name" type="String">rdr_fees</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="5" type="Map">
				<parameter name="Name" type="String">rdr_action</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="6" type="Map">
				<parameter name="Name" type="String">rdr_channel_id</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="7" type="Map">
				<parameter name="Name" type="String">rdr_acti</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="8" type="Map">
				<parameter name="Name" type="String">rdr_status</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="9" type="Map">
				<parameter name="Name" type="String">rdr_statusCode</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="10" type="Map">
				<parameter name="Name" type="String">rdr_log_type</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="11" type="Map">
				<parameter name="Name" type="String">rdr_error</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
		</parameter>
	</aim>
	<tasklist>
		<starttask class="wedo.ecc.im.reader.InputReader" format="1" name="Reader" taskkey="TaskInputReader" x="1125" y="2190">
			<parameter name="InitAllFields" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="BlockTypes" type="Map" vbtype="Map">
				<parameter name="2000008" type="Map">
					<parameter name="RolloverBlocks" type="Integer">-1</parameter>
					<parameter name="ReaderName" type="String">DSV Reader</parameter>
					<parameter name="ContinueOnError" type="Boolean">False</parameter>
					<parameter name="SendEmptyRecord" type="Boolean">False</parameter>
					<parameter name="DelayRollover" type="Boolean">False</parameter>
					<parameter name="SendEOBRecord" type="Boolean">False</parameter>
					<parameter name="SendErrorRecord" type="Boolean">False</parameter>
					<parameter name="ErrorsThreshold" type="Integer">1</parameter>
					<parameter name="UnlimitedErrors" type="Boolean">False</parameter>
					<parameter name="CommitOnRollover" type="Boolean">True</parameter>
					<parameter name="Schema" type="Map">
						<parameter name="Delimiter" type="String">,</parameter>
						<parameter name="EOLDelimiter" type="String"></parameter>
						<parameter name="GunzipContents" type="Boolean">False</parameter>
						<parameter name="RequireAllTokens" type="Boolean">False</parameter>
						<parameter name="InputFields" type="Xml">
							<inputfield fieldindex="0" fieldname="rdr_correlationId"></inputfield>
							<inputfield fieldindex="3" fieldname="rdr_MSISDN"></inputfield>
							<inputfield fieldindex="6" fieldname="rdr_TIMESTAMP"></inputfield>
							<inputfield fieldindex="8" fieldname="rdr_action"></inputfield>
							<inputfield fieldindex="9" fieldname="rdr_channel_id"></inputfield>
							<inputfield fieldindex="12" fieldname="rdr_fees"></inputfield>
							<inputfield fieldindex="11" fieldname="rdr_status"></inputfield>
							<inputfield fieldindex="10" fieldname="rdr_statusCode"></inputfield>
							<inputfield fieldindex="7" fieldname="rdr_log_type"></inputfield>
							<inputfield fieldindex="14" fieldname="rdr_error"></inputfield>
						</parameter>
					</parameter>
					<parameter name="FilterRules" type="Xml">
						<And completeeval="false">
							<Not>
								<IsEmpty field="rdr_TIMESTAMP"></IsEmpty>
							</Not>
							<Not>
								<StrMatch field="rdr_correlationId" search="*correlationId*"></StrMatch>
							</Not>
						</And>
					</parameter>
				</parameter>
			</parameter>
			<parameter name="ControlFields" type="Xml" vbtype="IXMLDOMNode"></parameter>
			<parameter name="ConstantFields" type="Xml" vbtype="IXMLDOMNode"></parameter>
			<parameter name="PersistentFields" type="Array" vbtype="Array"></parameter>
		</starttask>
		<task class="wedo.ecc.im.dump.Dumper" format="1" name="DUMP" taskkey="TaskDump" x="11250" y="2355">
			<parameter name="DestinationTable" type="String" vbtype="String">DJ_T_TIBCO</parameter>
			<parameter name="MaxEntries" type="Integer" vbtype="Long">50000</parameter>
			<parameter name="BatchSize" type="Integer" vbtype="Integer">2000</parameter>
			<parameter name="RollbackColumn" type="String" vbtype="String"></parameter>
			<parameter name="Hint" type="String" vbtype="String"></parameter>
			<parameter name="Fields" type="Xml" vbtype="IXMLDOMNode">
				<field attribute="" column="ACTION_NAME" expression="[action]"></field>
				<field attribute="" column="CHANNEL_ID" expression="[channel_id]"></field>
				<field attribute="" column="CHARGE" expression="[rdr_fees]"></field>
				<field attribute="" column="CORRELATION_ID" expression="[correlationId]"></field>
				<field attribute="" column="MONITORING_POINT" expression="[MONITORING_POINT]"></field>
				<field attribute="" column="MSISDN" expression="[MSISDN]"></field>
				<field attribute="" column="START_DATE" expression="[START_DATE]" formula="TO_DATE(:START_DATE, &apos;YYYYMMDD&apos;)"></field>
				<field attribute="" column="START_HOUR" expression="[START_HOUR]"></field>
				<field attribute="" column="START_TIME" expression="[START_TIME]" formula="TO_DATE(:START_TIME, &apos;YYYYMMDDHH24MISS&apos;)"></field>
				<field attribute="" column="STATUS" expression="[STATUS]"></field>
			</parameter>
			<parameter name="Module" type="String" vbtype="String"></parameter>
			<parameter name="System" type="String" vbtype="String"></parameter>
			<parameter name="Columns" type="Map" vbtype="Map"></parameter>
			<parameter name="ErrorTable" type="Map" vbtype="Map"></parameter>
			<parameter name="Entity" type="String" vbtype="String"></parameter>
			<parameter name="RollbackAttribute" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="1" name="TRANS" taskkey="TaskTransform" x="3975" y="-150">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">2</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Or completeeval="true">
					<RuleBlock>
						<Iif>
							<Or>
								<StrMatch field="rdr_correlationId" search="* "></StrMatch>
								<StrMatch field="rdr_correlationId" search=" *"></StrMatch>
							</Or>
							<False></False>
						</Iif>
					</RuleBlock>
					<RuleBlock>
						<RuleBlock>
							<Assign append="False" constant="TIBCO" target="MONITORING_POINT"></Assign>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_statusCode"></IsEmpty>
										<Equals constant="" target="rdr_statusCode"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_statusCode"></Equals>
									</Or>
								</Not>
								<Iif>
									<StrMatch field="rdr_statusCode" ignorecase="True" search="0"></StrMatch>
									<Assign append="False" constant="Success" target="STATUS"></Assign>
									<Assign append="False" constant="Fail" target="STATUS"></Assign>
								</Iif>
								<Assign append="False" constant="_N" target="STATUS"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_fees"></IsEmpty>
										<Equals constant="" target="rdr_fees"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_fees"></Equals>
									</Or>
								</Not>
								<RuleBlock>
									<StrReplace replace="" search="EGP" source="rdr_fees"></StrReplace>
									<StrReplace replace="" search=" " source="rdr_fees"></StrReplace>
								</RuleBlock>
								<Assign append="False" constant="0" target="rdr_fees"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_action"></IsEmpty>
										<Equals constant="" target="rdr_action"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_action"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_action" target="action"></Assign>
								<Assign append="False" constant="_N" target="action"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_channel_id"></IsEmpty>
										<Equals constant="" target="rdr_channel_id"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_channel_id"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_channel_id" target="channel_id"></Assign>
								<Assign append="False" constant="_N" target="channel_id"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_correlationId"></IsEmpty>
										<Equals constant="" target="rdr_correlationId"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_correlationId"></Equals>
									</Or>
								</Not>
								<And>
									<Iif>
										<StrMatch field="rdr_correlationId" ignorecase="true" search="*;*"></StrMatch>
										<And>
											<RegExpPart field="rdr_correlationId" group="1" pattern="(.*)\;(.*)" target="correlationId"></RegExpPart>
											<RegExpPart field="rdr_correlationId" group="2" pattern="(.*)\;(.*)" target="paymob_id"></RegExpPart>
										</And>
										<And>
											<Assign append="False" source="rdr_correlationId" target="correlationId"></Assign>
											<Assign append="False" constant="_N" target="paymob_id"></Assign>
										</And>
									</Iif>
								</And>
								<And>
									<Assign append="False" constant="_N" target="correlationId"></Assign>
									<Assign append="False" constant="_N" target="paymob_id"></Assign>
								</And>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_MSISDN"></IsEmpty>
										<Equals constant="" target="rdr_MSISDN"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_MSISDN"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_MSISDN" target="MSISDN"></Assign>
								<Assign append="False" constant="_N" target="MSISDN"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<LogMessage iserror="False" loglevel="100" message="Time is {0}">
								<field source="rdr_TIMESTAMP"></field>
							</LogMessage>
							<StrReplace replace="" search="-" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="/" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="T" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="+" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search=":" source="rdr_TIMESTAMP"></StrReplace>
							<SubString from="1" length="14" source="rdr_TIMESTAMP"></SubString>
							<SubString from="1" length="8" source="rdr_TIMESTAMP" target="START_DATE"></SubString>
							<SubString from="9" length="2" source="rdr_TIMESTAMP" target="START_HOUR"></SubString>
							<Assign append="False" source="rdr_TIMESTAMP" target="START_TIME"></Assign>
						</RuleBlock>
					</RuleBlock>
				</Or>
			</parameter>
		</task>
		<task class="wedo.ecc.im.aggreg.Aggregator" format="1" name="DUMP 2" taskkey="TaskAggregator" x="7755" y="1845">
			<parameter name="DestinationTable" type="String" vbtype="String">DJ_T_TIBCO_V2</parameter>
			<parameter name="MaxEntries" type="Integer" vbtype="Long">50000</parameter>
			<parameter name="BatchSize" type="Integer" vbtype="Integer">2000</parameter>
			<parameter name="SkipInserts" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="SkipUpdates" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="Dimensions" type="Xml" vbtype="IXMLDOMNode">
				<dimension attribute="" column="ACTION_NAME" expression="[action]" handlenulls="0"></dimension>
				<dimension attribute="" column="CHANNEL_ID" expression="[channel_id]" handlenulls="0"></dimension>
				<dimension attribute="" column="CHARGE" expression="[rdr_fees]" handlenulls="0"></dimension>
				<dimension attribute="" column="CORRELATION_ID" expression="[correlationId]" handlenulls="0"></dimension>
				<dimension attribute="" column="ERROR" expression="[ERROR]" handlenulls="0"></dimension>
				<dimension attribute="" column="MONITORING_POINT" expression="[MONITORING_POINT]" handlenulls="0"></dimension>
				<dimension attribute="" column="MSISDN" expression="[MSISDN]" handlenulls="0"></dimension>
				<dimension attribute="" column="PAYMOB_ID" expression="[paymob_id]" handlenulls="0"></dimension>
				<dimension attribute="" column="START_DATE" expression="[START_DATE]" formula="TO_DATE(:START_DATE, &apos;YYYYMMDD&apos;)" handlenulls="0"></dimension>
				<dimension attribute="" column="START_HOUR" expression="[START_HOUR]" handlenulls="0"></dimension>
				<dimension attribute="" column="START_TIME" expression="[START_TIME]" formula="TO_DATE(:START_TIME, &apos;YYYYMMDDHH24MISS&apos;)" handlenulls="0"></dimension>
				<dimension attribute="" column="STATUS" expression="[STATUS]" handlenulls="0"></dimension>
			</parameter>
			<parameter name="Measures" type="Xml" vbtype="IXMLDOMNode">
				<Count attribute="" childmergeformula="cboChildMergeFormula" column="TRX_COUNT" mergeformula="cboMergeFormula"></Count>
			</parameter>
			<parameter name="Module" type="String" vbtype="String"></parameter>
			<parameter name="System" type="String" vbtype="String"></parameter>
			<parameter name="Columns" type="Map" vbtype="Map"></parameter>
			<parameter name="Hint" type="String" vbtype="String"></parameter>
			<parameter name="ErrorTable" type="Map" vbtype="Map"></parameter>
			<parameter name="Entity" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="1" name="TRANS-2" taskkey="TaskTransform" x="4410" y="3000">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">3</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
				<parameter name="3" type="String">Output3</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Or completeeval="true">
					<RuleBlock>
						<Iif>
							<Or>
								<StrMatch field="rdr_correlationId" search="* "></StrMatch>
								<StrMatch field="rdr_correlationId" search=" *"></StrMatch>
							</Or>
							<False></False>
						</Iif>
					</RuleBlock>
					<RuleBlock>
						<RuleBlock>
							<Assign append="False" constant="TIBCO" target="MONITORING_POINT"></Assign>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_statusCode"></IsEmpty>
										<Equals constant="" target="rdr_statusCode"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_statusCode"></Equals>
									</Or>
								</Not>
								<Iif>
									<StrMatch field="rdr_statusCode" ignorecase="True" search="0"></StrMatch>
									<Assign append="False" constant="Success" target="STATUS"></Assign>
									<Assign append="False" constant="Fail" target="STATUS"></Assign>
								</Iif>
								<Assign append="False" constant="_N" target="STATUS"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_fees"></IsEmpty>
										<Equals constant="" target="rdr_fees"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_fees"></Equals>
									</Or>
								</Not>
								<RuleBlock>
									<StrReplace replace="" search="EGP" source="rdr_fees"></StrReplace>
									<StrReplace replace="" search=" " source="rdr_fees"></StrReplace>
								</RuleBlock>
								<Assign append="False" constant="0" target="rdr_fees"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_action"></IsEmpty>
										<Equals constant="" target="rdr_action"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_action"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_action" target="action"></Assign>
								<Assign append="False" constant="_N" target="action"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_channel_id"></IsEmpty>
										<Equals constant="" target="rdr_channel_id"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_channel_id"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_channel_id" target="channel_id"></Assign>
								<Assign append="False" constant="_N" target="channel_id"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_correlationId"></IsEmpty>
										<Equals constant="" target="rdr_correlationId"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_correlationId"></Equals>
									</Or>
								</Not>
								<And>
									<Iif>
										<StrMatch field="rdr_correlationId" ignorecase="true" search="*;*"></StrMatch>
										<And>
											<RegExpPart field="rdr_correlationId" group="1" pattern="(.*)\;(.*)" target="correlationId"></RegExpPart>
											<RegExpPart field="rdr_correlationId" group="2" pattern="(.*)\;(.*)" target="paymob_id"></RegExpPart>
										</And>
										<And>
											<Assign append="False" source="rdr_correlationId" target="correlationId"></Assign>
											<Assign append="False" constant="_N" target="paymob_id"></Assign>
										</And>
									</Iif>
								</And>
								<And>
									<Assign append="False" constant="_N" target="correlationId"></Assign>
									<Assign append="False" constant="_N" target="paymob_id"></Assign>
								</And>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_MSISDN"></IsEmpty>
										<Equals constant="" target="rdr_MSISDN"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_MSISDN"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_MSISDN" target="MSISDN"></Assign>
								<Assign append="False" constant="_N" target="MSISDN"></Assign>
							</Iif>
						</RuleBlock>
						<RuleBlock>
							<LogMessage iserror="False" loglevel="100" message="Time is {0}">
								<field source="rdr_TIMESTAMP"></field>
							</LogMessage>
							<StrReplace replace="" search="-" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="/" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="T" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search="+" source="rdr_TIMESTAMP"></StrReplace>
							<StrReplace replace="" search=":" source="rdr_TIMESTAMP"></StrReplace>
							<SubString from="1" length="14" source="rdr_TIMESTAMP"></SubString>
							<SubString from="1" length="8" source="rdr_TIMESTAMP" target="START_DATE"></SubString>
							<SubString from="9" length="2" source="rdr_TIMESTAMP" target="START_HOUR"></SubString>
							<Assign append="False" source="rdr_TIMESTAMP" target="START_TIME"></Assign>
						</RuleBlock>
                           	<RuleBlock>
                               <Length source="rdr_error" target="LEN_RDR_ERROR"></Length>
                           
                               <Iif>
                                   <GreaterThan field="LEN_RDR_ERROR" constant="700"></GreaterThan>
                           
                                   <SubString
                                       from="1"
                                       length="700"
                                       source="rdr_error"
                                       target="rdr_error">
                                   </SubString>
                               </Iif>
                            </RuleBlock>
						<RuleBlock>
							<Iif>
								<Not>
									<Or>
										<IsEmpty field="rdr_error"></IsEmpty>
										<Equals constant="" target="rdr_error"></Equals>
										<Equals constant="null" ignorecase="True" target="rdr_error"></Equals>
									</Or>
								</Not>
								<Assign append="False" source="rdr_error" target="ERROR"></Assign>
								<Assign append="False" constant="_N" target="ERROR"></Assign>
							</Iif>
						</RuleBlock>
					</RuleBlock>
					<RuleBlock>
						<Iif>
							<Equals constant="CI_LOGS" target="rdr_log_type"></Equals>
							<SetOutput output="1"></SetOutput>
						</Iif>
					</RuleBlock>
				</Or>
			</parameter>
		</task>
		<task class="wedo.ecc.im.aggreg.Aggregator" format="1" name="CI_logs" taskkey="TaskAggregator" x="7905" y="4350">
			<parameter name="DestinationTable" type="String" vbtype="String">DJ_TIBCO_CI_LOGS</parameter>
			<parameter name="MaxEntries" type="Integer" vbtype="Long">50000</parameter>
			<parameter name="BatchSize" type="Integer" vbtype="Integer">2000</parameter>
			<parameter name="SkipInserts" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="SkipUpdates" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="Dimensions" type="Xml" vbtype="IXMLDOMNode">
				<dimension attribute="" column="CHARGE" expression="[rdr_fees]" handlenulls="0"></dimension>
				<dimension attribute="" column="CORRELATION_ID" expression="[correlationId]" handlenulls="0"></dimension>
				<dimension attribute="" column="MONITORING_POINT" expression="[MONITORING_POINT]" handlenulls="0"></dimension>
				<dimension attribute="" column="MSISDN" expression="[MSISDN]" handlenulls="0"></dimension>
				<dimension attribute="" column="START_DATE" expression="[START_DATE]" formula="TO_DATE(:START_DATE, &apos;YYYYMMDD&apos;)" handlenulls="0"></dimension>
				<dimension attribute="" column="START_HOUR" expression="[START_HOUR]" handlenulls="0"></dimension>
				<dimension attribute="" column="START_TIME" expression="[START_TIME]" formula="TO_DATE(:START_TIME, &apos;YYYYMMDDHH24MISS&apos;)" handlenulls="0"></dimension>
				<dimension attribute="" column="STATUS" expression="[STATUS]" handlenulls="0"></dimension>
			</parameter>
			<parameter name="Measures" type="Xml" vbtype="IXMLDOMNode">
				<Count attribute="" childmergeformula="cboChildMergeFormula" column="TRX_COUNT" mergeformula="cboMergeFormula"></Count>
			</parameter>
			<parameter name="Module" type="String" vbtype="String"></parameter>
			<parameter name="System" type="String" vbtype="String"></parameter>
			<parameter name="Columns" type="Map" vbtype="Map"></parameter>
			<parameter name="Hint" type="String" vbtype="String"></parameter>
			<parameter name="ErrorTable" type="Map" vbtype="Map"></parameter>
			<parameter name="Entity" type="String" vbtype="String"></parameter>
		</task>
	</tasklist>
	<linklist>
		<link from="Reader" out="1" to="TRANS-2" x="3025" y="2910"></link>
		<link from="TRANS-2" out="1" to="DUMP 2" x="6310" y="3720"></link>
		<link from="TRANS-2" out="3" to="CI_logs" x="6310" y="4320"></link>
	</linklist>
</flow>"

now i will send you an example of xml to see the used xml version :  this is working but u use the past one which raidesd this error tell me what did u understood and
will u do and give:

<?xml version="1.0"?>
<flow name="LOAD_BSCS_PROFILES" version="1">
	<flowdescription valueof="LOAD_BSCS"></flowdescription>
	<engine class="wedo.bpm.services.engine.AssynchronousProcess">
		<parameter name="DatabaseTraceLevel" type="Integer">6</parameter>
		<parameter name="MemoryTraceLevel" type="Integer">6</parameter>
	</engine>
	<configuration>
		<AutoRefresh value="1"></AutoRefresh>
		<RefreshDelay value="30"></RefreshDelay>
		<ShowErrorMessages value="1"></ShowErrorMessages>
		<ShowInformationMessages value="1"></ShowInformationMessages>
		<ShowRunWindow value="1"></ShowRunWindow>
		<DefaultEntryName value=""></DefaultEntryName>
		<SecurityPermissions>
			<Visible></Visible>
			<Modify></Modify>
			<Execute></Execute>
		</SecurityPermissions>
	</configuration>
	<flowsharedarea>
		<parameter name="ALARM_CONTEXT_TYPE" type="String">ALARMS</parameter>
		<parameter name="ALARM_MODULE" type="String">RAID</parameter>
	</flowsharedarea>
	<aim>
		<parameter name="Name" type="String"></parameter>
		<parameter name="Description" type="String"></parameter>
		<parameter name="Elements" type="Array">
			<parameter name="1" type="Map">
				<parameter name="Name" type="String">BSCS_OWB_RESULT</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="2" type="Map">
				<parameter name="Name" type="String">BSCS_DUPLICATE_COUNT</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="3" type="Map">
				<parameter name="Name" type="String">BSCS_NEW_SUBSCRIBER_COUNT</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="4" type="Map">
				<parameter name="Name" type="String">BSCS_DELETED_SUBSCRIBER_COUNT</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="5" type="Map">
				<parameter name="Name" type="String">BSCS_NUM_BAD</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="6" type="Map">
				<parameter name="Name" type="String">BSCS_NUM_DB_ERROR</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="7" type="Map">
				<parameter name="Name" type="String">BSCS_NUM_SKIPPED</parameter>
				<parameter name="Type" type="String">Long</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="8" type="Map">
				<parameter name="Name" type="String">BSCS_STAGE_ID</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="9" type="Map">
				<parameter name="Name" type="String">BSCS_EXISTING_PLATFORM_DETAIL</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="10" type="Map">
				<parameter name="Name" type="String">BSCS_TRUNCATE_STAGING</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="11" type="Map">
				<parameter name="Name" type="String">BSCS_CREATE_NEW_VERSION</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="12" type="Map">
				<parameter name="Name" type="String">FINALISE</parameter>
				<parameter name="Type" type="String">Boolean</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">True</parameter>
			</parameter>
			<parameter name="13" type="Map">
				<parameter name="Name" type="String">LOAD_TYPE_ID</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">True</parameter>
			</parameter>
			<parameter name="14" type="Map">
				<parameter name="Name" type="String">VERSION_NO</parameter>
				<parameter name="Type" type="String">Integer</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="15" type="Map">
				<parameter name="Name" type="String">BSCS_CSV_FILE_STRING</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="16" type="Map">
				<parameter name="Name" type="String">BSCS_PREPROCESSOR_CMD</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="17" type="Map">
				<parameter name="Name" type="String">BSCS_PREPROCESSOR_ARGS</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="18" type="Map">
				<parameter name="Name" type="String">ERROR</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="19" type="Map">
				<parameter name="Name" type="String">TRUNCATE_STG</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="20" type="Map">
				<parameter name="Name" type="String">LOADING_STATUS</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="21" type="Map">
				<parameter name="Name" type="String">DESCRIPTION</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="22" type="Map">
				<parameter name="Name" type="String">FILES_STATUS</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="23" type="Map">
				<parameter name="Name" type="String">RUN_DATE</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="24" type="Map">
				<parameter name="Name" type="String">SIEBEL_LOADING_STATUS</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
			<parameter name="25" type="Map">
				<parameter name="Name" type="String">INS_RUN_DATE</parameter>
				<parameter name="Type" type="String">String</parameter>
				<parameter name="Description" type="String"></parameter>
				<parameter name="Mandatory" type="Boolean">False</parameter>
			</parameter>
		</parameter>
	</aim>
	<tasklist>
		<starttask class="wedo.bpm.task.TaskSemaphore" format="1" name="Start" taskkey="TaskSemaphore" x="90" y="270">
			<parameter name="name" type="String" vbtype="String">S_BSCS_PROFILES</parameter>
			<parameter name="value" type="String" vbtype="String">2</parameter>
			<parameter name="OpType" type="String" vbtype="String">S</parameter>
		</starttask>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="TRUNCATE_STG_TABLE" taskkey="TaskPlSql" x="105" y="4650">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.TRUNC_BSCS_PROFILE_TMP_STG(IN Number 1)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.ecc.im.services.core.LoaderTask" format="0" name="BSCS_Profiles" taskkey="IMLoader" x="4698" y="5190">
			<parameter name="ConnectionName" type="String" vbtype="String">RAIDPI2</parameter>
			<parameter name="Category" type="String" vbtype="String">Internal</parameter>
			<parameter name="OpType" type="Integer" vbtype="Long">0</parameter>
			<parameter name="Parameters" type="Map" vbtype="Map">
				<parameter name="JobId" type="String">25</parameter>
			</parameter>
			<parameter name="ErrorField" type="String" vbtype="String">ERROR</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="DUPS_INSERT" taskkey="TaskSql" x="3435" y="7971">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_PROFILE_DUPS
SELECT MSISDN,
  STATUS,
  IMSI,
  SERVICE_CODE,
  TYPE,
  CONTRACT_ID,
  TM_CODE,
  RATE_PLAN,
  TO_CHAR(EXTRACT_DATE,&apos;YYYYMMDD&apos;) AS RUN_DATE,
  BILLCYCLE,
  RUNNING_NUMBER
FROM PI_BSCS_TMP_STG_LDR
WHERE (MSISDN, IMSI) IN
  (SELECT MSISDN,
    IMSI
  FROM PI_BSCS_TMP_STG_LDR
  GROUP BY MSISDN,
    IMSI
  HAVING COUNT(1) &gt;1
  )</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.SetParameter" format="0" name="Set Params" taskkey="SetParameter" x="3210" y="3270">
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_PREPROCESSOR_CMD</parameter>
					<parameter name="value" type="String">echo</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_PREPROCESSOR_ARGS</parameter>
					<parameter name="value" type="String"></parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">PLATFORM</parameter>
					<parameter name="value" type="String">BSCS_PROFILE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">RUN_DATE</parameter>
					<parameter name="value" type="String"></parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSemaphore" format="0" name="Yellow" taskkey="TaskSemaphore" x="570" y="2415">
			<parameter name="name" type="String" vbtype="String">S_BSCS_PROFILES</parameter>
			<parameter name="value" type="String" vbtype="String">1</parameter>
			<parameter name="OpType" type="String" vbtype="String">U</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSemaphore" format="0" name="Green" taskkey="TaskSemaphore" x="13830" y="5265">
			<parameter name="name" type="String" vbtype="String">S_BSCS_PROFILES</parameter>
			<parameter name="value" type="String" vbtype="String">2</parameter>
			<parameter name="OpType" type="String" vbtype="String">U</parameter>
		</task>
		<task class="wedo.bpm.task.TaskTunnel" format="0" name="End" taskkey="TaskTunnel" x="16077" y="57"></task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Platform Detail Entry" taskkey="TaskPlSql" x="651" y="9147">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PKG_PI_UTIL_ADM.CREATE_PLATFORM_DETAIL(IN Varchar2 BSCS, IN Long [VERSION_NO], IN Number [LOAD_TYPE_ID]) RETURN Number</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">BSCS_EXISTING_PLATFORM_DETAIL</parameter>
					<parameter name="type" type="String">Integer</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="General Stats" taskkey="TaskSql" x="5151" y="9087">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">select PKG_PI_UTIL_ADM.GET_RECORD_COUNT(&apos;BSCS&apos;, &apos;[VERSION_NO]&apos;), PKG_PI_UTIL_ADM.GET_DUPLICATE_COUNT(&apos;BSCS&apos;, &apos;MSISDN&apos;, [VERSION_NO]), PKG_PI_UTIL_ADM.GET_NEW_SUBSCRIBER_COUNT(&apos;BSCS&apos;, [VERSION_NO]), PKG_PI_UTIL_ADM.GET_DELETED_SUBSCRIBER_COUNT(&apos;BSCS&apos;, [VERSION_NO]) from dual</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">BSCS_RECORD_COUNT</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="field" type="String">2</parameter>
					<parameter name="name" type="String">BSCS_DUPLICATE_COUNT</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="field" type="String">3</parameter>
					<parameter name="name" type="String">BSCS_NEW_SUBSCRIBER_COUNT</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="field" type="String">4</parameter>
					<parameter name="name" type="String">BSCS_DELETED_SUBSCRIBER_COUNT</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">True</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Save Stats" taskkey="TaskPlSql" x="8088" y="9108">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PKG_PI_UTIL_ADM.UPDATE_PLATFORM_DETAIL(IN Varchar2 BSCS, IN Long [VERSION_NO], IN Number [BSCS_STAGE_ID], IN Long [BSCS_RECORD_COUNT], IN Long [BSCS_DUPLICATE_COUNT], IN Long [BSCS_NUM_BAD], IN Long [BSCS_NUM_DB_ERROR], IN Long [BSCS_NUM_SKIPPED], IN Long [BSCS_NEW_SUBSCRIBER_COUNT], IN Long [BSCS_DELETED_SUBSCRIBER_COUNT])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.SetParameter" format="0" name="Set Stage Partial" taskkey="SetParameter" x="9468" y="9108">
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_STAGE_ID</parameter>
					<parameter name="value" type="String">14</parameter>
					<parameter name="type" type="String">Integer</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_NUM_BAD</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_NUM_DB_ERROR</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_NUM_SKIPPED</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_NEW_SUBSCRIBER_COUNT</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_RECORD_COUNT</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_DUPLICATE_COUNT</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_DELETED_SUBSCRIBER_COUNT</parameter>
					<parameter name="value" type="String">0</parameter>
					<parameter name="type" type="String">Long</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.SetParameter" format="0" name="Set Stage Complete" taskkey="SetParameter" x="6624" y="9081">
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_STAGE_ID</parameter>
					<parameter name="value" type="String">3</parameter>
					<parameter name="type" type="String">Integer</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskConditional" format="0" name="Check Finalise" taskkey="TaskConditional" x="8814" y="1761">
			<parameter name="name" type="String" vbtype="String">LOADING_STATUS</parameter>
			<parameter name="value" type="String" vbtype="String">1</parameter>
			<parameter name="type" type="String" vbtype="String">String</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="GET_RUN_DATE" taskkey="TaskSql" x="2157" y="9165">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT TO_CHAR(SYSDATE, &apos;&apos;YYYYMMDD&apos;&apos;) FROM DUAL</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">[RUN_DATE]</parameter>
					<parameter name="name" type="String">RUN_DATE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSendMail" format="0" name="NotifyFinish" taskkey="TaskSendMail" x="13875" y="7839">
			<parameter name="Configuration" type="Map" vbtype="String">
				<parameter name="Subject" type="Map">
					<parameter name="Template" type="String">PI [RUN_DATE]  -Loding BSCS Profiles Finished Successfully</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="HTML" type="Map">
					<parameter name="Template" type="String"></parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="Text" type="Map">
					<parameter name="Template" type="String">Dears,

Kindly be noted that the loading BSCS Profiles in PI [RUN_DATE] has finished successfully.

Regards,
PI Team</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="From" type="String">PI_RAID7@PI-Server</parameter>
				<parameter name="Reply" type="String">TechIT-BSApplicationSupportEnterprise@voda.com</parameter>
				<parameter name="RecipientsList" type="Array">
					<parameter name="1" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BSRevenueAssurance@voda.com</parameter>
					</parameter>
					<parameter name="2" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BI-ApplicationSupport@vodafone.com</parameter>
					</parameter>
				</parameter>
				<parameter name="UserNotificationFlag" type="String"></parameter>
				<parameter name="Attachements" type="Array"></parameter>
				<parameter name="InlineAttachements" type="Array"></parameter>
				<parameter name="Settings" type="Map">
					<parameter name="UseSystemDefaults" type="Boolean">False</parameter>
					<parameter name="Debug" type="Boolean">False</parameter>
					<parameter name="Server" type="String">10.230.95.91</parameter>
					<parameter name="ServerPort" type="String">25</parameter>
					<parameter name="Password" type="String"></parameter>
					<parameter name="Username" type="String"></parameter>
					<parameter name="UseAuthentication" type="Boolean">False</parameter>
					<parameter name="SSL" type="Boolean">False</parameter>
					<parameter name="TLS" type="Boolean">False</parameter>
					<parameter name="Fallback" type="Boolean">False</parameter>
				</parameter>
			</parameter>
			<parameter name="fromMail" type="String" vbtype="String"></parameter>
			<parameter name="mailServer" type="String" vbtype="String"></parameter>
			<parameter name="toMail" type="String" vbtype="String"></parameter>
			<parameter name="ccMail" type="String" vbtype="String"></parameter>
			<parameter name="bccMail" type="String" vbtype="String"></parameter>
			<parameter name="subject" type="String" vbtype="String"></parameter>
			<parameter name="message" type="String" vbtype="String"></parameter>
			<parameter name="attach" type="String" vbtype="String"></parameter>
			<parameter name="auth" type="String" vbtype="String">N</parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="0" name="Check_RUN_DATE" taskkey="TaskTransform" x="5823" y="2664">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">2</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Iif>
					<Or>
						<Equals constant="" ignorecase="True" target="RUN_DATE"></Equals>
						<IsEmpty field="RUN_DATE"></IsEmpty>
					</Or>
					<False></False>
					<True></True>
				</Iif>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSemaphore" format="0" name="Red" taskkey="TaskSemaphore" x="10830" y="3420">
			<parameter name="name" type="String" vbtype="String">S_BSCS_PROFILES</parameter>
			<parameter name="value" type="String" vbtype="String">0</parameter>
			<parameter name="OpType" type="String" vbtype="String">U</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="GET_LOADING_STATUS" taskkey="TaskSql" x="13980" y="8916">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT LOADING_STATUS FROM PI_R_LOADING_PLATFORM
WHERE PLATFORM = &apos;[PLATFORM]&apos;
AND RUN_DATE = &apos;[RUN_DATE]&apos;</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">LOADING_STATUS</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="GET_FILES_STATUS" taskkey="TaskSql" x="11184" y="10836">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT DES,STATUS FROM PI_R_PROCESSING_FILES 
WHERE PLATFORM = &apos;[PLATFORM]&apos;
AND RUN_DATE = [RUN_DATE]
AND ENTRY_DATE = ( 
SELECT MAX(ENTRY_DATE) 
FROM PI_R_PROCESSING_FILES 
WHERE PLATFORM = &apos;[PLATFORM]&apos;
AND RUN_DATE = [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">DES</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="field" type="String">2</parameter>
					<parameter name="name" type="String">STATUS</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSendMail" format="0" name="No_File_STATUS" taskkey="TaskSendMail" x="14433" y="11241">
			<parameter name="Configuration" type="Map" vbtype="String">
				<parameter name="Subject" type="Map">
					<parameter name="Template" type="String">PI [RUN_DATE] - BSCS Files Check Failed</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="HTML" type="Map">
					<parameter name="Template" type="String"></parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="Text" type="Map">
					<parameter name="Template" type="String">Dears,

Kindly be noted that the loading BSCS in PI [RUN_DATE] has stopped due to failing to check the files status.

Regards,
RA Team</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="From" type="String">PI@PI-Server</parameter>
				<parameter name="Reply" type="String">TechIT-BSApplicationSupportEnterprise@voda.com</parameter>
				<parameter name="RecipientsList" type="Array">
					<parameter name="1" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BSRevenueAssurance@voda.com</parameter>
					</parameter>
					<parameter name="2" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BI-ApplicationSupport@vodafone.com</parameter>
					</parameter>
				</parameter>
				<parameter name="UserNotificationFlag" type="String"></parameter>
				<parameter name="Attachements" type="Array"></parameter>
				<parameter name="InlineAttachements" type="Array"></parameter>
				<parameter name="Settings" type="Map">
					<parameter name="UseSystemDefaults" type="Boolean">False</parameter>
					<parameter name="Debug" type="Boolean">False</parameter>
					<parameter name="Server" type="String">10.230.95.91</parameter>
					<parameter name="ServerPort" type="String">25</parameter>
					<parameter name="Password" type="String"></parameter>
					<parameter name="Username" type="String"></parameter>
					<parameter name="UseAuthentication" type="Boolean">False</parameter>
					<parameter name="SSL" type="Boolean">False</parameter>
					<parameter name="TLS" type="Boolean">False</parameter>
					<parameter name="Fallback" type="Boolean">False</parameter>
				</parameter>
			</parameter>
			<parameter name="fromMail" type="String" vbtype="String"></parameter>
			<parameter name="mailServer" type="String" vbtype="String"></parameter>
			<parameter name="toMail" type="String" vbtype="String"></parameter>
			<parameter name="ccMail" type="String" vbtype="String"></parameter>
			<parameter name="bccMail" type="String" vbtype="String"></parameter>
			<parameter name="subject" type="String" vbtype="String"></parameter>
			<parameter name="message" type="String" vbtype="String"></parameter>
			<parameter name="attach" type="String" vbtype="String"></parameter>
			<parameter name="auth" type="String" vbtype="String">N</parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="0" name="CHECK_FILES_STATUS" taskkey="TaskTransform" x="12699" y="10644">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">2</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Iif>
					<Equals constant="1" ignorecase="True" target="STATUS"></Equals>
					<True></True>
					<False></False>
				</Iif>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSendMail" format="0" name="Files_NoGO_STATUS" taskkey="TaskSendMail" x="12786" y="11757">
			<parameter name="Configuration" type="Map" vbtype="String">
				<parameter name="Subject" type="Map">
					<parameter name="Template" type="String">PI [RUN_DATE] - BSCS Files Check Failed</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="HTML" type="Map">
					<parameter name="Template" type="String"></parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="Text" type="Map">
					<parameter name="Template" type="String">Dears,

Kindly be noted that the loading BSCS in PI RUN [RUN_DATE] has stopped due to the below error.

Regards,
RA Team</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="From" type="String">PI@PI-Server</parameter>
				<parameter name="Reply" type="String">TechIT-BSApplicationSupportEnterprise@voda.com</parameter>
				<parameter name="RecipientsList" type="Array">
					<parameter name="1" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BSRevenueAssurance@voda.com</parameter>
					</parameter>
					<parameter name="2" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BI-ApplicationSupport@vodafone.com</parameter>
					</parameter>
				</parameter>
				<parameter name="UserNotificationFlag" type="String"></parameter>
				<parameter name="Attachements" type="Array"></parameter>
				<parameter name="InlineAttachements" type="Array"></parameter>
				<parameter name="Settings" type="Map">
					<parameter name="UseSystemDefaults" type="Boolean">False</parameter>
					<parameter name="Debug" type="Boolean">False</parameter>
					<parameter name="Server" type="String">10.230.95.91</parameter>
					<parameter name="ServerPort" type="String">25</parameter>
					<parameter name="Password" type="String"></parameter>
					<parameter name="Username" type="String"></parameter>
					<parameter name="UseAuthentication" type="Boolean">False</parameter>
					<parameter name="SSL" type="Boolean">False</parameter>
					<parameter name="TLS" type="Boolean">False</parameter>
					<parameter name="Fallback" type="Boolean">False</parameter>
				</parameter>
			</parameter>
			<parameter name="fromMail" type="String" vbtype="String"></parameter>
			<parameter name="mailServer" type="String" vbtype="String"></parameter>
			<parameter name="toMail" type="String" vbtype="String"></parameter>
			<parameter name="ccMail" type="String" vbtype="String"></parameter>
			<parameter name="bccMail" type="String" vbtype="String"></parameter>
			<parameter name="subject" type="String" vbtype="String"></parameter>
			<parameter name="message" type="String" vbtype="String"></parameter>
			<parameter name="attach" type="String" vbtype="String"></parameter>
			<parameter name="auth" type="String" vbtype="String">N</parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.bpm.task.JobTask" format="0" name="RunFilesCheck" taskkey="JobTask" x="9690" y="10821">
			<parameter name="cmdline" type="String" vbtype="String">/apps/raidprd/PI_Scripts/BSCS_Profiles_files_check.sh</parameter>
			<parameter name="parameters" type="String" vbtype="String">[RUN_DATE]</parameter>
			<parameter name="StopValues" type="String" vbtype="String"></parameter>
			<parameter name="ContinueValues" type="String" vbtype="String"></parameter>
			<parameter name="resultName" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="RemoveDupsfromSTG" taskkey="TaskSql" x="4923" y="7971">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">DELETE FROM PI_BSCS_TMP_STG_LDR
WHERE (MSISDN, IMSI) IN (SELECT MSISDN, IMSI 
FROM PFM_BSCS_PROFILE_DUPS 
WHERE RUN_DATE = [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="INSERT_PLATFORM" taskkey="TaskSql" x="3675" y="9144">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS
SELECT * FROM PFM_BSCS_STG</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="UPDATE_PLATFORM_STATUS" taskkey="TaskSql" x="15648" y="8820">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO 
PI_R_LOADING_PLATFORM (PLATFORM,RUN_DATE,LOADING_STATUS) 
VALUES (&apos;BSCS_PROFILE&apos;,&apos;[RUN_DATE]&apos;,&apos;1&apos;)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="CREATE_PART" taskkey="TaskPlSql" x="1863" y="5544">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.ADD_PLATFORM_PARTITION(IN Varchar2 PFM_BSCS_PROFILE, IN Varchar2 [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="ADSL_DUPS_INSERT" taskkey="TaskSql" x="9198" y="7899">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_ADSL_DUPS
SELECT * FROM PFM_BSCS_ADSL_STG
WHERE MSISDN in (
SELECT MSISDN FROM (
SELECT MSISDN ,COUNT(1) AS CUST_COUNT FROM PFM_BSCS_ADSL_STG
GROUP BY MSISDN)
WHERE CUST_COUNT &gt;1)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="ADSL_REMOVEDUP" taskkey="TaskSql" x="10791" y="7932">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">DELETE FROM PFM_BSCS_ADSL_STG
WHERE MSISDN IN (SELECT MSISDN 
FROM PFM_BSCS_ADSL_DUPS
WHERE RUN_DATE = [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="GET_RUN_DATE_PL" taskkey="TaskPlSql" x="4515" y="1830">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.GET_RUN_DATE(IN Varchar2 0) RETURN Varchar2</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">RUN_DATE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="INSERT_ADSL" taskkey="TaskSql" x="12384" y="7899">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_ADSL
SELECT * FROM PFM_BSCS_ADSL_STG</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="ALTER_INDEX_2" taskkey="TaskPlSql" x="7776" y="7944">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">RP7VFEPIDAT.ALTER_BSCS_INDEX(IN Number 4)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="SIEBEL_SERVICE_STATUS" taskkey="TaskSql" x="14403" y="10263">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT sum(LOADING_STATUS) STATUS_ALL 
FROM PI_R_LOADING_PLATFORM
WHERE PLATFORM in ( &apos;SIEBEL&apos;)
AND RUN_DATE = &apos;[RUN_DATE]&apos;</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">STATUS_ALL</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="0" name="SET_RUN_DATE" taskkey="TaskTransform" x="1995" y="2190">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">4</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
				<parameter name="3" type="String">RunDateSet</parameter>
				<parameter name="4" type="String">RunDatenotSet</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Iif>
					<Or>
						<IsEmpty field="RUN_DATE"></IsEmpty>
						<Equals constant="" ignorecase="False" target="RUN_DATE"></Equals>
					</Or>
					<SetOutput output="2"></SetOutput>
					<SetOutput output="1"></SetOutput>
				</Iif>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="INSERT_PLATFORM_NEW" taskkey="TaskPlSql" x="6300" y="8001">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">RP7VFEPIDAT.INSERT_BSCS_PROFILE(IN Varchar2 [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.SetParameter" format="0" name="SET_RUN_DATE1" taskkey="SetParameter" x="3255" y="1230">
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_PREPROCESSOR_CMD</parameter>
					<parameter name="value" type="String">echo</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">BSCS_PREPROCESSOR_ARGS</parameter>
					<parameter name="value" type="String"></parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">PLATFORM</parameter>
					<parameter name="value" type="String">BSCS_PROFILE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
				<parameter type="Map">
					<parameter name="name" type="String">RUN_DATE</parameter>
					<parameter name="value" type="String">[RUN_DATE]</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.ecc.im.transform.Transform" format="0" name="check Services-Seibel" taskkey="TaskTransform" x="15996" y="10218">
			<parameter name="outputNumber" type="Integer" vbtype="Integer">2</parameter>
			<parameter name="OutputNames" type="Array" vbtype="Array">
				<parameter name="1" type="String">True</parameter>
				<parameter name="2" type="String">False</parameter>
			</parameter>
			<parameter name="MultipleOutputs" type="Boolean" vbtype="boolean">False</parameter>
			<parameter name="LoggerService" type="String" vbtype="String"></parameter>
			<parameter name="Rules" type="Xml" vbtype="IXMLDOMNode">
				<Or>
					<Iif>
						<Equals constant="1" ignorecase="True" target="STATUS_ALL"></Equals>
						<True></True>
						<False></False>
					</Iif>
				</Or>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskError" format="0" name="Error" taskkey="TaskError" x="10779" y="6243">
			<parameter name="Message" type="String" vbtype="String">[ERROR]</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSemaphore" format="0" name="RED2" taskkey="TaskSemaphore" x="7299" y="6231">
			<parameter name="name" type="String" vbtype="String">S_BSCS_PROFILES</parameter>
			<parameter name="value" type="String" vbtype="String">0</parameter>
			<parameter name="OpType" type="String" vbtype="String">U</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="ALTER_STG" taskkey="TaskPlSql" x="2043" y="7983">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">RP7VFEPIDAT.ALTER_BSCS_INDEX(IN Number 3)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="CREATE_ADSL_PART" taskkey="TaskPlSql" x="3204" y="5541">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.ADD_PLATFORM_PARTITION(IN Varchar2 PFM_BSCS_ADSL, IN Varchar2 [RUN_DATE])</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="Insert_ADSL_D" taskkey="TaskSql" x="10959" y="9099">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_ADSL
SELECT MSISDN,
  STATUS,
  CASE
    WHEN BILLCYCLE = &apos;03&apos;
    THEN &apos;PRE&apos;
    ELSE &apos;POST&apos;
  END AS TYPE,
  CONTRACT_ID,
  TM_CODE,
  TO_CHAR(EXTRACT_DATE,&apos;YYYYMMDD&apos;) AS RUN_DATE,
BILLCYCLE
FROM PI_BSCS_TMP_STG_LDR
WHERE SERVICE_CODE = 504</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="Insert_Platform2" taskkey="TaskSql" x="15495" y="7767">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_PROFILE
SELECT MSISDN,
  STATUS,
  IMSI,
  SERVICE_CODE,
  TYPE,
  CONTRACT_ID,
  TM_CODE,
  RATE_PLAN,
  TO_CHAR(EXTRACT_DATE,&apos;YYYYMMDD&apos;) AS RUN_DATE,
  BILLCYCLE,
  RUNNING_NUMBER
FROM PI_BSCS_TMP_STG_LDR_PROFILE</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="Insert_ADSL_STG" taskkey="TaskSql" x="12411" y="9051">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">INSERT INTO PFM_BSCS_ADSL_STG
SELECT MSISDN,
  STATUS,
  CASE
    WHEN BILLCYCLE = &apos;03&apos;
    THEN &apos;PRE&apos;
    ELSE &apos;POST&apos;
  END AS TYPE,
  CONTRACT_ID,
  TM_CODE,
  TO_CHAR(EXTRACT_DATE,&apos;YYYYMMDD&apos;) AS RUN_DATE,
BILLCYCLE
FROM PI_BSCS_TMP_STG_LDR
WHERE SERVICE_CODE = 504</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">False</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="Get_DAILY_RUN_DATE" taskkey="TaskSql" x="702" y="8130">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT MAX(RUN_DATE) AS RUN_DATE FROM PI_RUN_DATE WHERE RUN_TYPE = &apos;DAILY&apos;</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">RUN_DATE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">True</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Exchange_Partition" taskkey="TaskPlSql" x="7281" y="5013">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.EXCHANGE_PARTITION(IN Varchar2 [RUN_DATE], IN Varchar2 PFM_BSCS_PROFILE, IN Varchar2 PI_BSCS_TMP_STG_LDR_PROFILE)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Get_Loading_Status_PL" taskkey="TaskPlSql" x="7392" y="1722">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.GET_LOADING_STATUS(IN Varchar2 [RUN_DATE], IN Varchar2 [PLATFORM]) RETURN Varchar2</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">LOADING_STATUS</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Get_Siebel_Loading_Status_PL" taskkey="TaskPlSql" x="10734" y="1323">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.GET_LOADING_STATUS(IN Varchar2 [RUN_DATE], IN Varchar2 SIEBEL) RETURN Varchar2</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">SIEBEL_LOADING_STATUS</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
		</task>
		<task class="wedo.bpm.task.TaskConditional" format="0" name="Check_Siebel_Status" taskkey="TaskConditional" x="12735" y="1035">
			<parameter name="name" type="String" vbtype="String">SIEBEL_LOADING_STATUS</parameter>
			<parameter name="value" type="String" vbtype="String">0</parameter>
			<parameter name="type" type="String" vbtype="String">String</parameter>
		</task>
		<task class="wedo.bpm.task.TaskPlSql" format="0" name="Insert_Loading_Status" taskkey="TaskPlSql" x="10725" y="5016">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="plsql" type="String" vbtype="String">PI_UTIL_PKG.INSERT_LOADING_STATUS(IN Varchar2 [RUN_DATE], IN Varchar2 BSCS_PROFILE)</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSql" format="0" name="Check_Data" taskkey="TaskSql" x="8994" y="5028">
			<parameter name="driver" type="String" vbtype="String"></parameter>
			<parameter name="url" type="String" vbtype="String"></parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
			<parameter name="statement" type="String" vbtype="String">SELECT DISTINCT RUN_DATE AS INS_RUN_DATE FROM PFM_BSCS_ADSL_STG</parameter>
			<parameter name="connection" type="String" vbtype="String"></parameter>
			<parameter name="returnConfig" type="Array" vbtype="Array">
				<parameter type="Map">
					<parameter name="field" type="String">1</parameter>
					<parameter name="name" type="String">INS_RUN_DATE</parameter>
					<parameter name="type" type="String">String</parameter>
				</parameter>
			</parameter>
			<parameter name="iterator" type="Boolean" vbtype="Boolean">True</parameter>
		</task>
		<task class="wedo.bpm.task.TaskSendMail" format="0" name="Notify_Failed" taskkey="TaskSendMail" x="8943" y="6252">
			<parameter name="Configuration" type="Map" vbtype="String">
				<parameter name="Subject" type="Map">
					<parameter name="Template" type="String">PI [RUN_DATE]  -Loding BSCS Profiles Failed</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="HTML" type="Map">
					<parameter name="Template" type="String"></parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="Text" type="Map">
					<parameter name="Template" type="String">Dears,

Kindly be noted that the loading BSCS Profiles in PI [RUN_DATE] has failed,

[ERROR]

Regards,
PI Team</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="From" type="String">PI_RAID7@vodafone.com</parameter>
				<parameter name="Reply" type="String">TechIT-BSApplicationSupportEnterprise@voda.com</parameter>
				<parameter name="RecipientsList" type="Array">
					<parameter name="1" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BSRevenueAssurance@voda.com</parameter>
					</parameter>
					<parameter name="2" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BI-ApplicationSupport@vodafone.com</parameter>
					</parameter>
				</parameter>
				<parameter name="UserNotificationFlag" type="String"></parameter>
				<parameter name="Attachements" type="Array"></parameter>
				<parameter name="InlineAttachements" type="Array"></parameter>
				<parameter name="Settings" type="Map">
					<parameter name="UseSystemDefaults" type="Boolean">False</parameter>
					<parameter name="Debug" type="Boolean">False</parameter>
					<parameter name="Server" type="String">10.230.30.32</parameter>
					<parameter name="ServerPort" type="String">25</parameter>
					<parameter name="Password" type="String"></parameter>
					<parameter name="Username" type="String"></parameter>
					<parameter name="UseAuthentication" type="Boolean">False</parameter>
					<parameter name="SSL" type="Boolean">False</parameter>
					<parameter name="TLS" type="Boolean">False</parameter>
					<parameter name="Fallback" type="Boolean">False</parameter>
				</parameter>
			</parameter>
			<parameter name="fromMail" type="String" vbtype="String"></parameter>
			<parameter name="mailServer" type="String" vbtype="String"></parameter>
			<parameter name="toMail" type="String" vbtype="String"></parameter>
			<parameter name="ccMail" type="String" vbtype="String"></parameter>
			<parameter name="bccMail" type="String" vbtype="String"></parameter>
			<parameter name="subject" type="String" vbtype="String"></parameter>
			<parameter name="message" type="String" vbtype="String"></parameter>
			<parameter name="attach" type="String" vbtype="String"></parameter>
			<parameter name="auth" type="String" vbtype="String">N</parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
		</task>
		<task class="wedo.bpm.task.TaskSendMail" format="0" name="Notify_Failed2" taskkey="TaskSendMail" x="12324" y="3027">
			<parameter name="Configuration" type="Map" vbtype="String">
				<parameter name="Subject" type="Map">
					<parameter name="Template" type="String">PI ADSL [RUN_DATE]  -Loading BSCS Profiles Failed</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="HTML" type="Map">
					<parameter name="Template" type="String"></parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="Text" type="Map">
					<parameter name="Template" type="String">Dears,

Kindly be noted that the loading BSCS Profiles in PI [RUN_DATE] has failed,

Regards,
PI Team</parameter>
					<parameter name="TemplateType" type="String">1</parameter>
				</parameter>
				<parameter name="From" type="String">PI_RAID7@vodafone.com</parameter>
				<parameter name="Reply" type="String">TechIT-BSApplicationSupportEnterprise@voda.com</parameter>
				<parameter name="RecipientsList" type="Array">
					<parameter name="1" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BSRevenueAssurance@voda.com</parameter>
					</parameter>
					<parameter name="2" type="Map">
						<parameter name="Recipient" type="String">TO</parameter>
						<parameter name="Type" type="String">EMAIL</parameter>
						<parameter name="Targets" type="String">TechIT-BI-ApplicationSupport@vodafone.com</parameter>
					</parameter>
				</parameter>
				<parameter name="UserNotificationFlag" type="String"></parameter>
				<parameter name="Attachements" type="Array"></parameter>
				<parameter name="InlineAttachements" type="Array"></parameter>
				<parameter name="Settings" type="Map">
					<parameter name="UseSystemDefaults" type="Boolean">False</parameter>
					<parameter name="Debug" type="Boolean">False</parameter>
					<parameter name="Server" type="String">10.230.30.32</parameter>
					<parameter name="ServerPort" type="String">25</parameter>
					<parameter name="Password" type="String"></parameter>
					<parameter name="Username" type="String"></parameter>
					<parameter name="UseAuthentication" type="Boolean">False</parameter>
					<parameter name="SSL" type="Boolean">False</parameter>
					<parameter name="TLS" type="Boolean">False</parameter>
					<parameter name="Fallback" type="Boolean">False</parameter>
				</parameter>
			</parameter>
			<parameter name="fromMail" type="String" vbtype="String"></parameter>
			<parameter name="mailServer" type="String" vbtype="String"></parameter>
			<parameter name="toMail" type="String" vbtype="String"></parameter>
			<parameter name="ccMail" type="String" vbtype="String"></parameter>
			<parameter name="bccMail" type="String" vbtype="String"></parameter>
			<parameter name="subject" type="String" vbtype="String"></parameter>
			<parameter name="message" type="String" vbtype="String"></parameter>
			<parameter name="attach" type="String" vbtype="String"></parameter>
			<parameter name="auth" type="String" vbtype="String">N</parameter>
			<parameter name="user" type="String" vbtype="String"></parameter>
			<parameter name="password" type="String" vbtype="String"></parameter>
		</task>
	</tasklist>
	<linklist>
		<link from="Start" out="3" to="Yellow" x="1990" y="1575"></link>
		<link from="Start" out="1" to="End" x="1990" y="975"></link>
		<link from="Start" out="2" to="End" x="1990" y="1230"></link>
		<link from="TRUNCATE_STG_TABLE" out="1" to="CREATE_PART" x="2005" y="5481"></link>
		<link from="BSCS_Profiles" out="2" to="RED2" x="5998" y="6231"></link>
		<link from="BSCS_Profiles" out="1" to="Exchange_Partition" x="5998" y="5925"></link>
		<link from="DUPS_INSERT" out="1" to="RemoveDupsfromSTG" x="4735" y="8706"></link>
		<link from="DUPS_INSERT" out="2" to="RemoveDupsfromSTG" x="4735" y="9006"></link>
		<link from="Set Params" out="1" to="GET_RUN_DATE_PL" x="4510" y="4065"></link>
		<link from="Yellow" out="2" to="SET_RUN_DATE" x="1870" y="3360"></link>
		<link from="Green" out="3" to="End" x="15130" y="6150"></link>
		<link from="General Stats" out="1" to="Set Stage Complete" x="6451" y="9522"></link>
		<link from="Check Finalise" out="1" to="Green" x="10114" y="2520"></link>
		<link from="Check Finalise" out="2" to="TRUNCATE_STG_TABLE" x="10114" y="2781"></link>
		<link from="Check_RUN_DATE" out="2" to="Red" x="7123" y="3684"></link>
		<link from="Check_RUN_DATE" out="1" to="Get_Loading_Status_PL" x="7123" y="3375"></link>
		<link from="Red" out="1" to="Notify_Failed2" x="12130" y="4119"></link>
		<link from="GET_FILES_STATUS" out="1" to="CHECK_FILES_STATUS" x="12484" y="11541"></link>
		<link from="GET_FILES_STATUS" out="2" to="Files_NoGO_STATUS" x="12484" y="11796"></link>
		<link from="CHECK_FILES_STATUS" out="1" to="SIEBEL_SERVICE_STATUS" x="13999" y="11274"></link>
		<link from="CHECK_FILES_STATUS" out="2" to="No_File_STATUS" x="13999" y="11694"></link>
		<link from="RunFilesCheck" out="1" to="GET_FILES_STATUS" x="10990" y="11601"></link>
		<link from="CREATE_PART" out="1" to="CREATE_ADSL_PART" x="3163" y="6291"></link>
		<link from="ADSL_DUPS_INSERT" out="1" to="ADSL_REMOVEDUP" x="10498" y="8664"></link>
		<link from="ADSL_DUPS_INSERT" out="2" to="ADSL_REMOVEDUP" x="10498" y="8874"></link>
		<link from="ADSL_REMOVEDUP" out="1" to="INSERT_ADSL" x="12091" y="8697"></link>
		<link from="ADSL_REMOVEDUP" out="2" to="INSERT_ADSL" x="12091" y="9012"></link>
		<link from="GET_RUN_DATE_PL" out="1" to="Check_RUN_DATE" x="5815" y="2673"></link>
		<link from="SIEBEL_SERVICE_STATUS" out="1" to="check Services-Seibel" x="15703" y="10983"></link>
		<link from="SET_RUN_DATE" out="4" to="Set Params" x="3295" y="3735"></link>
		<link from="SET_RUN_DATE" out="3" to="SET_RUN_DATE1" x="3295" y="3480"></link>
		<link from="SET_RUN_DATE1" out="1" to="Get_Loading_Status_PL" x="4555" y="1953"></link>
		<link from="Error" out="1" to="End" x="12079" y="7023"></link>
		<link from="RED2" out="1" to="Notify_Failed" x="8599" y="7002"></link>
		<link from="CREATE_ADSL_PART" out="1" to="BSCS_Profiles" x="4504" y="6264"></link>
		<link from="Exchange_Partition" out="1" to="Check_Data" x="8581" y="5688"></link>
		<link from="Get_Loading_Status_PL" out="1" to="Check Finalise" x="8692" y="2481"></link>
		<link from="Check_Siebel_Status" out="1" to="TRUNCATE_STG_TABLE" x="14035" y="1806"></link>
		<link from="Check_Siebel_Status" out="2" to="Green" x="14035" y="2040"></link>
		<link from="Insert_Loading_Status" out="1" to="Green" x="12025" y="5655"></link>
		<link from="Check_Data" out="1" to="Insert_Loading_Status" x="10294" y="5787"></link>
		<link from="Check_Data" out="2" to="Red" x="10294" y="6045"></link>
		<link from="Notify_Failed" out="1" to="Error" x="10243" y="6975"></link>
		<link from="Notify_Failed2" out="1" to="End" x="13624" y="3690"></link>
	</linklist>
</flow>
