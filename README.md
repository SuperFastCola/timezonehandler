Usage:


```
#!php

function checkIfClientAlreadySubmittedAnEntry(){

		//converts UTC time in DB to EST/EDT
		//gets server time converted to EST/EDT
		//sets entry and servre time to midnight
		//then check if both times are the same timestamp

		//check for last record matching email address
		$sqlstatement = $this->db->query("Select timestamp from eh_entries where email regexp '" . $this->entry['email'] . "' order by id desc limit 1");
		if($sqlstatement->num_rows>0){
			//start time zone object
			if(!class_exists('TimeZoneHandler')){
				require 'lib/TimeZoneHandler.php';
			}

			$tz = new TimeZoneHandler($this->cf_timezone);
			$server_time = $tz->getServerTimeAdjusted(false);

			//entry date is stored in Database as UTC
			$entrydate = strtotime($sqlstatement->fetch_object()->timestamp);

			//current day for server time adjusted to EST at midnight
			$today_at_midnight = strtotime(date('Y-m-d', $server_time->server_adjusted->timestamp));

			//get offset between New York and UTC
			$tz_offset = $tz->getTimeOffsetBetween('UTC','America/New_York');

			//just in case timezones are switched subtract or add milliseconds to entry date
			if($tz_offset->hours<0){
				//milliseconds will be a minus number (-3600)
				$entrydate_converted_est = $entrydate + $tz_offset->milliseconds;
			}
			//if servertime hours is greater than utc time
			elseif($tz_offset->hours>0){
				//milliseconds will be a positive number (-3600)
				$entrydate_converted_est = $entrydate - $tz_offset->milliseconds;
			}
			else{
				$entrydate_converted_est = $entrydate;
			}

			//set converted timestamp to midnight EST
			$entry_at_midnight = strtotime(date('Y-m-d',$entrydate_converted_est));

			//error_log(date('Y-m-d H', $entry_at_midnight) . " " . date('Y-m-d H', $today_at_midnight));

			if($entry_at_midnight < $today_at_midnight){
				//error_log("Not Entered Today");
				return false;
			}
			else{
				//error_log("Entered Today");
				return true;
			}
		}
		else{
			return false;
		}
	}
```
