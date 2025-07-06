// Load NASA GPM IMERG Monthly precipitation dataset
var dataset = ee.ImageCollection('NASA/GPM_L3/IMERG_MONTHLY_V06');

// Define Iran bounding box polygon
var customRegion = ee.Geometry.Polygon(
  [[[44.0, 39.8],
    [44.0, 24.0],
    [63.3, 24.0],
    [63.3, 39.8]]]
);

// Function to pad the month with leading zero
var padMonth = function(month) {
  return month < 10 ? '0' + month : '' + month;
};

// Function to export the 'precipitation' band as GeoTIFF
var exportImageToTiff = function(image, year, month) {
  var dateString = year + '_' + padMonth(month);
  var precipitation = image.select('precipitation');

  Export.image.toDrive({
    image: precipitation,
    description: 'IMERG_MonthlyPrecipitation_' + dateString,
    region: customRegion,
    crs: 'EPSG:4326',
    scale: 0.01 * 11132, // ~1.1 km resolution
    maxPixels: 1e13
  });
};

// Loop over years and months (example for 2002)
for (var year = 2002; year <= 2002; year++) {
  for (var month = 1; month <= 12; month++) {
    var startDate = year + '-' + padMonth(month) + '-01';
    var startDateObj = ee.Date(startDate);
    var endDateObj = startDateObj.advance(1, 'month');

    var filteredDataset = dataset.filterDate(startDateObj, endDateObj);

    var image = filteredDataset.first();
    if (image) {
      exportImageToTiff(image, year, month);
    }
  }
}

print('Export process started. Check the "Tasks" tab for progress.');
