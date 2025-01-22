def monitor_temperatures(temp_readings: list[list[float]], warning_threshold: float, consecutive_hours: int) -> list[dict]:
    alerts = []
   
    for server_id, server_temps in enumerate(temp_readings):
        current_sequence = []
        sequence_start = None
        has_reported = False
       
        for hour, temp in enumerate(server_temps):
            # Skip invalid readings
            if temp is None:
                # Only reset sequence if we haven't met consecutive_hours requirement
                if len(current_sequence) < consecutive_hours:
                    current_sequence = []
                    sequence_start = None
                    has_reported = False
                continue
               
            if temp > warning_threshold:
                if not current_sequence:
                    sequence_start = hour
                current_sequence.append(temp)
               
                # Create or update alert once we hit the consecutive hours threshold
                if len(current_sequence) >= consecutive_hours and not has_reported:
                    alerts.append({
                        'server_id': server_id,
                        'start_hour': sequence_start,
                        'max_temp': max(current_sequence),
                        'duration': len(current_sequence)
                    })
                    has_reported = True
                # Update existing alert if we've already reported this sequence
                elif has_reported:
                    alerts[-1]['max_temp'] = max(alerts[-1]['max_temp'], temp)
                    alerts[-1]['duration'] = len(current_sequence)
            else:
                current_sequence = []
                sequence_start = None
                has_reported = False
               
    return alerts
