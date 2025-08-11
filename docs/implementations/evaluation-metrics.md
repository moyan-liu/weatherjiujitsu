# Evaluation Metrics: Measuring Weather Jiu-Jitsu Success

Quantifying the effectiveness of weather steering interventions requires comprehensive metrics that assess both intended outcomes and unintended consequences. This guide establishes standardized evaluation frameworks for different applications of Weather Jiu-Jitsu.

## Core Evaluation Principles

### Success Criteria Hierarchy
1. **Primary Objectives**: Direct impact on target weather system
2. **Secondary Effects**: Regional climate and ecosystem impacts  
3. **System Integrity**: Preservation of forecast skill and natural patterns
4. **Operational Metrics**: Cost-effectiveness and implementation feasibility

### Evaluation Timeline
- **Immediate (0-24h)**: Perturbation propagation and initial response
- **Short-term (1-7 days)**: Primary objective achievement
- **Medium-term (1-4 weeks)**: Secondary effects and system adjustment
- **Long-term (1-12 months)**: Cumulative impacts and pattern stability

## Tropical Cyclone Steering Metrics

### Track Modification Assessment
Primary metric for TC steering effectiveness.

```python
class TCSteeringMetrics:
    def __init__(self, reference_track, perturbed_track):
        self.reference_track = reference_track
        self.perturbed_track = perturbed_track
        
    def calculate_track_deviation(self):
        """
        Calculate spatial deviation between reference and perturbed tracks
        
        Returns:
            Track deviation metrics in km
        """
        deviations = []
        
        for ref_point, pert_point in zip(self.reference_track, self.perturbed_track):
            # Calculate great circle distance between track points
            deviation_km = self.haversine_distance(
                ref_point['lat'], ref_point['lon'],
                pert_point['lat'], pert_point['lon']
            )
            deviations.append(deviation_km)
        
        return {
            'max_deviation_km': np.max(deviations),
            'mean_deviation_km': np.mean(deviations),
            'final_deviation_km': deviations[-1],
            'deviation_timeline': deviations,
            'cumulative_track_error': np.sum(deviations)
        }
    
    def assess_landfall_modification(self, target_coastline):
        """
        Evaluate changes in landfall timing and location
        
        Args:
            target_coastline: Coastline polygon for landfall detection
        
        Returns:
            Landfall modification metrics
        """
        ref_landfall = self.detect_landfall(self.reference_track, target_coastline)
        pert_landfall = self.detect_landfall(self.perturbed_track, target_coastline)
        
        if ref_landfall and pert_landfall:
            distance_change = self.haversine_distance(
                ref_landfall['lat'], ref_landfall['lon'],
                pert_landfall['lat'], pert_landfall['lon']
            )
            time_change = (pert_landfall['time'] - ref_landfall['time']).total_seconds() / 3600
            
            return {
                'landfall_modified': True,
                'position_change_km': distance_change,
                'timing_change_hours': time_change,
                'avoided_landfall': False
            }
        elif ref_landfall and not pert_landfall:
            return {
                'landfall_modified': True,
                'position_change_km': np.inf,
                'timing_change_hours': np.inf,
                'avoided_landfall': True
            }
        else:
            return {
                'landfall_modified': False,
                'position_change_km': 0,
                'timing_change_hours': 0,
                'avoided_landfall': False
            }
    
    def evaluate_intensity_preservation(self):
        """
        Assess whether steering preserves TC intensity characteristics
        
        Returns:
            Intensity preservation metrics
        """
        ref_intensities = [point['intensity'] for point in self.reference_track]
        pert_intensities = [point['intensity'] for point in self.perturbed_track]
        
        intensity_correlation = np.corrcoef(ref_intensities, pert_intensities)[0,1]
        mean_intensity_change = np.mean(np.array(pert_intensities) - np.array(ref_intensities))
        max_intensity_change = np.max(np.abs(np.array(pert_intensities) - np.array(ref_intensities)))
        
        return {
            'intensity_correlation': intensity_correlation,
            'mean_intensity_change_kt': mean_intensity_change,
            'max_intensity_change_kt': max_intensity_change,
            'intensity_preserved': intensity_correlation > 0.8 and max_intensity_change < 10
        }
```

### Population Impact Metrics
Quantify human and economic consequences of track changes.

```python
def calculate_population_impact_change(reference_track, perturbed_track, 
                                     population_density_grid, wind_threshold=64):
    """
    Assess change in population exposure to dangerous winds
    
    Args:
        reference_track, perturbed_track: TC track data
        population_density_grid: Gridded population data
        wind_threshold: Wind speed threshold for impact (kt)
    
    Returns:
        Population impact change metrics
    """
    # Calculate wind fields for both tracks
    ref_wind_field = calculate_tc_wind_field(reference_track)
    pert_wind_field = calculate_tc_wind_field(perturbed_track)
    
    # Identify areas exceeding wind threshold
    ref_impact_area = ref_wind_field > wind_threshold
    pert_impact_area = pert_wind_field > wind_threshold
    
    # Calculate population exposure
    ref_exposed_population = np.sum(population_density_grid[ref_impact_area])
    pert_exposed_population = np.sum(population_density_grid[pert_impact_area])
    
    population_reduction = ref_exposed_population - pert_exposed_population
    percentage_reduction = (population_reduction / ref_exposed_population) * 100
    
    return {
        'exposed_population_change': population_reduction,
        'percentage_exposure_reduction': percentage_reduction,
        'reference_exposed_population': ref_exposed_population,
        'perturbed_exposed_population': pert_exposed_population,
        'impact_area_change_km2': np.sum(pert_impact_area) - np.sum(ref_impact_area)
    }
```

## Atmospheric River Modification Metrics

### Integrated Vapor Transport (IVT) Assessment
Core metric for AR intensity and track modifications.

```python
class ARModificationMetrics:
    def __init__(self, reference_ar_data, modified_ar_data):
        self.reference_ar = reference_ar_data
        self.modified_ar = modified_ar_data
    
    def calculate_ivt_modification(self):
        """
        Assess changes in AR moisture transport characteristics
        
        Returns:
            IVT modification metrics
        """
        # Calculate IVT fields
        ref_ivt = self.calculate_ivt_field(self.reference_ar)
        mod_ivt = self.calculate_ivt_field(self.modified_ar)
        
        # Core AR corridor analysis
        ref_corridor = self.identify_ar_corridor(ref_ivt, threshold=250)
        mod_corridor = self.identify_ar_corridor(mod_ivt, threshold=250)
        
        # IVT statistics
        ref_max_ivt = np.max(ref_ivt)
        mod_max_ivt = np.max(mod_ivt)
        
        ref_mean_ivt = np.mean(ref_ivt[ref_corridor])
        mod_mean_ivt = np.mean(mod_ivt[mod_corridor])
        
        # Spatial characteristics
        ref_corridor_area = np.sum(ref_corridor) * self.grid_cell_area
        mod_corridor_area = np.sum(mod_corridor) * self.grid_cell_area
        
        return {
            'max_ivt_change_percent': ((mod_max_ivt - ref_max_ivt) / ref_max_ivt) * 100,
            'mean_ivt_change_percent': ((mod_mean_ivt - ref_mean_ivt) / ref_mean_ivt) * 100,
            'corridor_area_change_km2': mod_corridor_area - ref_corridor_area,
            'total_moisture_transport_change': np.sum(mod_ivt) - np.sum(ref_ivt),
            'intensity_reduction_achieved': (ref_max_ivt - mod_max_ivt) > 0
        }
    
    def assess_precipitation_redistribution(self, target_region):
        """
        Evaluate how AR modifications affect regional precipitation
        
        Args:
            target_region: Geographic bounds for precipitation analysis
        
        Returns:
            Precipitation redistribution metrics
        """
        ref_precip = self.calculate_ar_precipitation(self.reference_ar, target_region)
        mod_precip = self.calculate_ar_precipitation(self.modified_ar, target_region)
        
        # Regional precipitation changes
        region_precip_change = np.sum(mod_precip - ref_precip)
        max_local_change = np.max(np.abs(mod_precip - ref_precip))
        
        # Spatial redistribution analysis
        precip_centroid_ref = self.calculate_precipitation_centroid(ref_precip)
        precip_centroid_mod = self.calculate_precipitation_centroid(mod_precip)
        
        centroid_shift_km = self.haversine_distance(
            precip_centroid_ref['lat'], precip_centroid_ref['lon'],
            precip_centroid_mod['lat'], precip_centroid_mod['lon']
        )
        
        return {
            'total_precipitation_change_mm': region_precip_change,
            'max_local_change_mm': max_local_change,
            'precipitation_centroid_shift_km': centroid_shift_km,
            'beneficial_redistribution': self.assess_beneficial_redistribution(
                ref_precip, mod_precip, target_region
            )
        }
```

### Flood Risk Assessment
Quantify changes in flood potential from AR modifications.

```python
def calculate_flood_risk_change(reference_ar, modified_ar, watershed_data):
    """
    Assess change in flood risk from AR modification
    
    Args:
        reference_ar, modified_ar: AR precipitation data
        watershed_data: Watershed characteristics and flood thresholds
    
    Returns:
        Flood risk change metrics
    """
    flood_risk_changes = {}
    
    for watershed_id, watershed_info in watershed_data.items():
        # Extract precipitation over watershed
        ref_watershed_precip = extract_watershed_precipitation(
            reference_ar, watershed_info['boundary']
        )
        mod_watershed_precip = extract_watershed_precipitation(
            modified_ar, watershed_info['boundary']
        )
        
        # Calculate flood probability using intensity-duration-frequency curves
        ref_flood_prob = calculate_flood_probability(
            ref_watershed_precip, watershed_info['idf_curves']
        )
        mod_flood_prob = calculate_flood_probability(
            mod_watershed_precip, watershed_info['idf_curves']
        )
        
        flood_risk_changes[watershed_id] = {
            'flood_probability_change': mod_flood_prob - ref_flood_prob,
            'peak_precipitation_change': np.max(mod_watershed_precip) - np.max(ref_watershed_precip),
            'flood_risk_reduced': mod_flood_prob < ref_flood_prob,
            'risk_reduction_percent': ((ref_flood_prob - mod_flood_prob) / ref_flood_prob) * 100 if ref_flood_prob > 0 else 0
        }
    
    return flood_risk_changes
```

## Blocking Pattern Disruption Metrics

### Pattern Persistence Assessment
Measure success in reducing blocking pattern duration and intensity.

```python
class BlockingDisruptionMetrics:
    def __init__(self, reference_fields, modified_fields):
        self.reference_fields = reference_fields
        self.modified_fields = modified_fields
    
    def calculate_blocking_index_change(self):
        """
        Assess changes in blocking pattern strength using standard indices
        
        Returns:
            Blocking index modification metrics
        """
        # Calculate Tibaldi-Molteni blocking index
        ref_blocking_index = self.calculate_tibaldi_molteni_index(self.reference_fields)
        mod_blocking_index = self.calculate_tibaldi_molteni_index(self.modified_fields)
        
        # Calculate Rex blocking index
        ref_rex_index = self.calculate_rex_blocking_index(self.reference_fields)
        mod_rex_index = self.calculate_rex_blocking_index(self.modified_fields)
        
        return {
            'tibaldi_molteni_change': mod_blocking_index - ref_blocking_index,
            'rex_index_change': mod_rex_index - ref_rex_index,
            'blocking_intensity_reduced': mod_blocking_index < ref_blocking_index,
            'blocking_disruption_percent': ((ref_blocking_index - mod_blocking_index) / ref_blocking_index) * 100 if ref_blocking_index != 0 else 0
        }
    
    def assess_pattern_longevity(self, forecast_days=14):
        """
        Evaluate changes in blocking pattern persistence
        
        Args:
            forecast_days: Number of days to assess pattern evolution
        
        Returns:
            Pattern longevity metrics
        """
        # Track blocking pattern over time
        ref_persistence = self.track_pattern_persistence(
            self.reference_fields, forecast_days
        )
        mod_persistence = self.track_pattern_persistence(
            self.modified_fields, forecast_days
        )
        
        persistence_reduction = ref_persistence - mod_persistence
        
        return {
            'reference_persistence_days': ref_persistence,
            'modified_persistence_days': mod_persistence,
            'persistence_reduction_days': persistence_reduction,
            'early_breakdown_achieved': persistence_reduction > 2,
            'persistence_reduction_percent': (persistence_reduction / ref_persistence) * 100 if ref_persistence > 0 else 0
        }
```

### Extreme Weather Impact Reduction
Quantify reduction in temperature and precipitation extremes.

```python
def calculate_extreme_weather_reduction(reference_fields, modified_fields, 
                                      impact_region, variable='temperature'):
    """
    Assess reduction in extreme weather conditions
    
    Args:
        reference_fields, modified_fields: Atmospheric field data
        impact_region: Geographic region for impact assessment
        variable: 'temperature', 'precipitation', or 'wind'
    
    Returns:
        Extreme weather reduction metrics
    """
    ref_values = extract_regional_values(reference_fields[variable], impact_region)
    mod_values = extract_regional_values(modified_fields[variable], impact_region)
    
    if variable == 'temperature':
        # Focus on heat wave reduction
        ref_extreme_days = np.sum(ref_values > np.percentile(ref_values, 95))
        mod_extreme_days = np.sum(mod_values > np.percentile(ref_values, 95))
        
        extreme_metric = 'extreme_heat_days_reduced'
        reduction = ref_extreme_days - mod_extreme_days
        
    elif variable == 'precipitation':
        # Focus on drought alleviation
        ref_dry_days = np.sum(ref_values < 1.0)  # Less than 1mm/day
        mod_dry_days = np.sum(mod_values < 1.0)
        
        extreme_metric = 'drought_days_reduced'
        reduction = ref_dry_days - mod_dry_days
    
    return {
        extreme_metric: reduction,
        'mean_value_change': np.mean(mod_values) - np.mean(ref_values),
        'extreme_conditions_improved': reduction > 0,
        'improvement_percentage': (reduction / len(ref_values)) * 100
    }
```

## System-Wide Validation Metrics

### Forecast Skill Preservation
Ensure weather modifications don't degrade forecast accuracy in other regions.

```python
class ForecastSkillValidation:
    def __init__(self, control_regions):
        self.control_regions = control_regions
    
    def assess_skill_degradation(self, reference_forecast, modified_forecast, 
                               observations):
        """
        Evaluate forecast skill changes in non-target regions
        
        Args:
            reference_forecast: Unperturbed forecast
            modified_forecast: Forecast with weather modification
            observations: Verification data
        
        Returns:
            Forecast skill comparison metrics
        """
        skill_changes = {}
        
        for region_name, region_bounds in self.control_regions.items():
            # Extract regional data
            ref_regional = extract_regional_forecast(reference_forecast, region_bounds)
            mod_regional = extract_regional_forecast(modified_forecast, region_bounds)
            obs_regional = extract_regional_observations(observations, region_bounds)
            
            # Calculate skill scores
            ref_rmse = calculate_rmse(ref_regional, obs_regional)
            mod_rmse = calculate_rmse(mod_regional, obs_regional)
            
            ref_correlation = calculate_correlation(ref_regional, obs_regional)
            mod_correlation = calculate_correlation(mod_regional, obs_regional)
            
            skill_changes[region_name] = {
                'rmse_change': mod_rmse - ref_rmse,
                'correlation_change': mod_correlation - ref_correlation,
                'skill_degraded': mod_rmse > ref_rmse or mod_correlation < ref_correlation,
                'skill_preservation_score': self.calculate_skill_preservation_score(
                    ref_rmse, mod_rmse, ref_correlation, mod_correlation
                )
            }
        
        return skill_changes
```

### Energy Conservation Assessment
Verify that perturbations don't violate fundamental physical principles.

```python
def verify_energy_conservation(perturbation_fields, atmospheric_state):
    """
    Verify that weather modifications conserve total atmospheric energy
    
    Args:
        perturbation_fields: Applied atmospheric perturbations
        atmospheric_state: Background atmospheric state
    
    Returns:
        Energy conservation validation results
    """
    # Calculate kinetic energy changes
    kinetic_energy_change = calculate_kinetic_energy_change(
        perturbation_fields['u'], perturbation_fields['v'], atmospheric_state
    )
    
    # Calculate potential energy changes
    potential_energy_change = calculate_potential_energy_change(
        perturbation_fields['temperature'], atmospheric_state
    )
    
    # Calculate total energy change
    total_energy_change = kinetic_energy_change + potential_energy_change
    
    # Energy conservation threshold (should be small)
    conservation_threshold = 1e-6  # Joules/kg
    
    return {
        'kinetic_energy_change': kinetic_energy_change,
        'potential_energy_change': potential_energy_change,
        'total_energy_change': total_energy_change,
        'energy_conserved': abs(total_energy_change) < conservation_threshold,
        'conservation_violation_magnitude': abs(total_energy_change),
        'conservation_score': np.exp(-abs(total_energy_change) / conservation_threshold)
    }
```

## Comprehensive Evaluation Framework

### Multi-Objective Assessment
Combine multiple metrics into overall success evaluation.

```python
class WeatherJiuJitsuEvaluator:
    def __init__(self, application_type):
        self.application_type = application_type
        self.metric_weights = self.load_metric_weights(application_type)
    
    def comprehensive_evaluation(self, reference_data, modified_data, 
                               target_objectives):
        """
        Perform comprehensive evaluation of weather modification intervention
        
        Args:
            reference_data: Baseline weather data
            modified_data: Weather data with modifications
            target_objectives: Intended outcomes of intervention
        
        Returns:
            Comprehensive evaluation results with overall success score
        """
        evaluation_results = {}
        
        # Application-specific primary metrics
        if self.application_type == 'tropical_cyclone_steering':
            primary_metrics = self.evaluate_tc_steering(
                reference_data, modified_data, target_objectives
            )
        elif self.application_type == 'atmospheric_river_modification':
            primary_metrics = self.evaluate_ar_modification(
                reference_data, modified_data, target_objectives
            )
        elif self.application_type == 'blocking_pattern_disruption':
            primary_metrics = self.evaluate_blocking_disruption(
                reference_data, modified_data, target_objectives
            )
        
        evaluation_results['primary_objectives'] = primary_metrics
        
        # System-wide validation metrics
        system_metrics = self.evaluate_system_impacts(reference_data, modified_data)
        evaluation_results['system_validation'] = system_metrics
        
        # Calculate weighted overall score
        overall_score = self.calculate_weighted_score(
            primary_metrics, system_metrics, self.metric_weights
        )
        
        evaluation_results['overall_success_score'] = overall_score
        evaluation_results['intervention_recommended'] = overall_score > 0.7
        
        # Generate detailed report
        evaluation_results['detailed_report'] = self.generate_evaluation_report(
            evaluation_results
        )
        
        return evaluation_results
    
    def calculate_weighted_score(self, primary_metrics, system_metrics, weights):
        """
        Calculate weighted composite score from multiple metric categories
        
        Args:
            primary_metrics: Application-specific success metrics
            system_metrics: System-wide validation metrics
            weights: Relative importance weights for different metric categories
        
        Returns:
            Composite success score (0-1 scale)
        """
        # Normalize all metrics to 0-1 scale
        normalized_primary = self.normalize_metrics(primary_metrics)
        normalized_system = self.normalize_metrics(system_metrics)
        
        # Calculate weighted average
        primary_score = np.mean(list(normalized_primary.values()))
        system_score = np.mean(list(normalized_system.values()))
        
        composite_score = (
            weights['primary_objectives'] * primary_score +
            weights['system_validation'] * system_score
        )
        
        return composite_score
```

## Operational Metrics and Reporting

### Real-Time Monitoring Dashboard
Track intervention effectiveness during operational deployment.

```python
class OperationalMonitoring:
    def __init__(self, intervention_config):
        self.intervention_config = intervention_config
        self.monitoring_start_time = datetime.now()
        
    def generate_real_time_metrics(self, current_observations, forecast_data):
        """
        Generate real-time assessment of intervention progress
        
        Args:
            current_observations: Latest observational data
            forecast_data: Current forecast with interventions
        
        Returns:
            Real-time intervention status metrics
        """
        elapsed_time = (datetime.now() - self.monitoring_start_time).total_seconds() / 3600
        
        # Track intervention timeline
        timeline_metrics = self.assess_intervention_timeline(
            elapsed_time, self.intervention_config
        )
        
        # Compare with expected outcomes
        expectation_metrics = self.compare_with_expectations(
            current_observations, forecast_data, elapsed_time
        )
        
        # Early warning for unintended effects
        warning_metrics = self.check_unintended_effects(
            current_observations, forecast_data
        )
        
        return {
            'timeline_status': timeline_metrics,
            'expectation_alignment': expectation_metrics,
            'warning_indicators': warning_metrics,
            'intervention_on_track': all([
                timeline_metrics['on_schedule'],
                expectation_metrics['meeting_expectations'],
                not warning_metrics['warnings_present']
            ])
        }
```

---

*Comprehensive evaluation is essential for establishing Weather Jiu-Jitsu as a reliable climate adaptation technology. The metrics and frameworks presented here provide the foundation for rigorous assessment of intervention effectiveness while ensuring responsible deployment that preserves atmospheric system integrity.*