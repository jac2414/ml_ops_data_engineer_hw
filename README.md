# Data Engineer Homework

### 1. Sensor Data Missing and Pipeline Robustness
 
#### Explanation and Response to Qs:
If one of the sensors was missing some data, this pipeline is designed to handle such scenarios gracefully. Here’s how:
 
- **Handling NaN Values**: The pipeline includes steps to fill NaN values using interpolation (`df['value'].interpolate(method='linear')`) for the `value` column, which assumes that sensor readings change linearly over time. For other columns like `time`, `robot_id`, and `run_uuid`, we use forward fill (`fillna(method='ffill')`) and backward fill (`fillna(method='bfill')`) methods, which propagate the last valid observation forward and backward respectively.
- **Assumptions**: The pipeline does assume a certain structure for the data, specifically that the data contains columns like `time`, `value`, `field`, `robot_id`, `run_uuid`, and `sensor_type`. It also assumes that the `field` values for `encoder` are among ['x', 'y', 'z'] and for `load_cell` are among ['fx', 'fy', 'fz'].
- **Accommodating Changes**: If there are changes or mistakes downstream, such as inconsistent `field` values or invalid `robot_id` or `sensor_type`, the pipeline includes checks to drop these inconsistent rows. Additionally, the outlier detection step identifies and can handle outliers.
 
#### Judgement on Cleansing Steps:
- **Linear Interpolation**: This is suitable if sensor readings are expected to change gradually over time.
- **Forward/Backward Fill**: This is appropriate when missing data points should reasonably be assumed to be similar to the preceding or following values.
- **Dropping Inconsistent Rows**: Ensures that only valid and expected data structures are processed further.
 
### 2. Choice of Fill Method for `df_filled`
 
#### Explanation:
During the step where `df_filled` is first defined, I used the forward fill (`fillna(method='ffill')`) and backward fill (`fillna(method='bfill')`) methods.
 
#### Benefits and Trade-offs:
- **Benefits**:
  - **Simplicity**: These methods are straightforward and ensure that no gaps remain in the data, which is essential for subsequent calculations like velocity and acceleration.
  - **Consistency**: They maintain consistency in the data by filling missing values with the most recent valid observations.
- **Trade-offs**:
  - **Assumption of Continuity**: These methods assume that the missing data points are similar to the nearest available data points. This might not always be true, especially if there are abrupt changes.
  - **Potential Bias**: Forward filling can introduce bias if there are long sequences of missing data, as it will replicate the last known value over an extended period.
 
#### Alternative Methods:
- **Interpolation**: Another option is linear interpolation (`df_grouped.interpolate(method='linear')`), which can provide a more accurate estimate for missing values if the data changes gradually over time.
- **Machine Learning Models**: More sophisticated methods could involve using machine learning models to predict missing values based on patterns in the data.
 
### 3. Parallelization and Sequential Steps in the Pipeline
 
#### Explanation:
Certain parts of the pipeline can be parallelized, while others need to be done sequentially due to dependencies between steps.
 
#### Parallelizable Parts:
- **Reading Data**: Reading the Parquet file and converting it to a DataFrame can be parallelized if there are multiple files or chunks of data.
- **Data Type Checks**: Converting columns to the expected data types can be done in parallel for different columns.
- **Consistency Checks**: Checking for invalid `robot_id` and `sensor_type` values can be done in parallel.
 
#### Sequential Parts:
- **NaN Handling**: The steps to handle NaN values (e.g., interpolation, forward/backward fill) need to be done sequentially as they depend on the order of data points.
- **Pivoting and Filling**: Pivoting the DataFrame and filling NaN values must be done sequentially as they transform the structure of the data.
- **Feature Engineering**: Calculating velocity, acceleration, and other derived features must be done sequentially as each step depends on the previous calculations.
 
