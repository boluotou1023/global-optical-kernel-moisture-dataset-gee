/*******************************
 * kNDMI/kSVMI Monthly Dataset Export (Scheme A) - FIXED snow QA
 * Output:
 *   INDEX_YYYY_MM.tif (Float32, multiband): kNDMI_005_fixed, kNDMI_020_max, kSVMI, snow_fraction
 *   QA_YYYY_MM.tif    (UInt16, multiband): valid_mask, qa_reason, n_valid_obs
 *******************************/

Map.setCenter(12.1, 39, 1);

// ====== User settings ======
var year = 2024; // change year here
var exportScale  = 1000;

var exportCRS    = 'EPSG:4326';
var exportRegion = table.geometry().bounds();

// If your dataset starts from April 2000:
var startMonth = (year === 2000) ? 4 : 1;
var endMonth   = 12;

// ====== Collections ======
var mod09a1_0 = ee.ImageCollection("MODIS/061/MOD09A1");
var mod09a1 = mod09a1_0.map(function(img){
  var clipped = img.clipToCollection(table);
  return clipped.copyProperties(img, ['system:time_start','system:time_end','system:index']);
});

var mod10a1 = ee.ImageCollection('MODIS/061/MOD10A1');

// MOD09A1 reflectance bands
var modisBands = ['sur_refl_b03','sur_refl_b04','sur_refl_b01','sur_refl_b02','sur_refl_b05','sur_refl_b06','sur_refl_b07'];
var IsBands    = ['blue','green','red','nir1','swir1','swir2','swir3'];

// ===============================
// 1) QA decode for MOD09A1 StateQA
// ===============================
function decodeStateQA(image) {
  var qa = image.select('StateQA');
  var cloudState = qa.bitwiseAnd(3);               // bits 0-1
  var isCloud  = cloudState.eq(1).or(cloudState.eq(2)); // cloudy or mixed
  var isShadow = qa.bitwiseAnd(1 << 2).neq(0);     // bit 2
  return ee.Image.cat([
    isCloud.rename('cloudBad').toByte(),
    isShadow.rename('shadowBad').toByte()
  ]);
}

// ===============================
// 2) Prepare MOD09A1 (range + cloud/shadow + scaling) and attach aux bands
// ===============================
function prepareMod09(image) {
  var reflRaw = image.select(modisBands);

  // rangeBad = 1 if ANY band out of [-100, 16000]
  var rangeBad = reflRaw.lt(-100).or(reflRaw.gt(16000))
    .reduce(ee.Reducer.anyNonZero()).rename('rangeBad').toByte();

  var qa = decodeStateQA(image);
  var cloudBad  = qa.select('cloudBad');
  var shadowBad = qa.select('shadowBad');

  // goodObs = in-range AND not cloud AND not shadow
  var goodObs = rangeBad.not().and(cloudBad.not()).and(shadowBad.not())
    .rename('goodObs').toByte();

  // scaled reflectance (0.0001) + mask goodObs
  var refl = reflRaw.updateMask(goodObs).divide(10000).select(modisBands, IsBands);

  // attach aux bands
  var aux = rangeBad.addBands(cloudBad).addBands(shadowBad).addBands(goodObs);

  return refl.addBands(aux)
    .copyProperties(image, ['system:time_start','system:time_end','system:index']);
}

// ===============================
// 3) Monthly snow summary from MOD10A1 (FIXED)
// Outputs:
//   snowMaskNoSnow: 1 (no snow), 0 (snow present in month)
//   snow_fraction: snow-day fraction in month (0-1)
// ===============================
function monthlySnowSummary(start, end) {
  var snowCol = mod10a1.filterDate(start, end)
    .select(['NDSI_Snow_Cover','NDSI_Snow_Cover_Basic_QA']);

  var has = snowCol.size().gt(0);

  return ee.Image(ee.Algorithms.If(has, (function() {

    // Per-day valid mask and snow-day mask (as ImageCollection of 0/1 images)
    var validDayCol = snowCol.map(function(im){
      var qa = im.select('NDSI_Snow_Cover_Basic_QA');
      var valid = qa.lte(2); // 0/1/2 usable
      return valid.rename('validDay').toByte()
        .copyProperties(im, ['system:time_start']);
    });

    var snowDayCol = snowCol.map(function(im){
      var ndsi = im.select('NDSI_Snow_Cover');
      var qa = im.select('NDSI_Snow_Cover_Basic_QA');
      var valid = qa.lte(2);
      var snow = ndsi.gte(1).and(ndsi.lte(100)).and(valid);
      return snow.rename('snowDay').toByte()
        .copyProperties(im, ['system:time_start']);
    });

    var snowDays  = snowDayCol.sum();
    var validDays = validDayCol.sum();

    // avoid division by zero
    validDays = validDays.where(validDays.eq(0), 1);

    var snowFrac = snowDays.divide(validDays).rename('snow_fraction').toFloat();

    // mask rule: ANY snow day => snow present
    var snowPresent = snowDays.gt(0);
    var snowMaskNoSnow = snowPresent.not().rename('snowMaskNoSnow').toByte();

    return snowMaskNoSnow.addBands(snowFrac);

  })(),
  ee.Image(1).rename('snowMaskNoSnow').toByte()
    .addBands(ee.Image(0).rename('snow_fraction').toFloat())
  ));
}

// ==========================================================
// 4) Index functions (DO NOT MODIFY)
// ==========================================================
function calkSVMI_005(img) {
  var blue  = img.select("blue");
  var red   = img.select("red");
  var swir2 = img.select("swir2");
  var swir3 = img.select("swir3");
  var xigema = ee.Image(0.05);

  var BR  = ((blue.subtract(red)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);
  var BS2 = ((blue.subtract(swir2)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);
  var BS3 = ((blue.subtract(swir3)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);
  var RS2 = ((red.subtract(swir2)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);
  var RS3 = ((red.subtract(swir3)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);
  var SS  = ((swir2.subtract(swir3)).pow(2)).divide(2).divide(xigema.pow(2)).multiply(-1);

  var kSVMI31_1 = (ee.Image(12)
    .subtract(ee.Image(6).multiply(img.expression('exp(b)', {'b': BR})))
    .subtract(ee.Image(6).multiply(img.expression('exp(b)', {'b': BS2})))
    .subtract(ee.Image(6).multiply(img.expression('exp(b)', {'b': BS3})))
    .add(ee.Image(2).multiply(img.expression('exp(b)', {'b': RS2})))
    .add(ee.Image(2).multiply(img.expression('exp(b)', {'b': RS3})))
    .add(ee.Image(2).multiply(img.expression('exp(b)', {'b': SS}))));

  var middle_sgn = ee.Image(3).multiply(blue).subtract(swir2).subtract(swir3).subtract(red);
  var kSVMI31_2 = middle_sgn.where(middle_sgn.gt(0), 1).where(middle_sgn.lt(0), -1);
  var unmasked_kSVMI31 = kSVMI31_1.multiply(kSVMI31_2);
  var kSVMI = unmasked_kSVMI31.add(ee.Image(18)).divide(ee.Image(36));
  return kSVMI.rename("kSVMI");
}

function calkNDMI_02_max(img) {
  var blue  = img.select("blue");
  var red   = img.select("red");
  var swir2 = img.select("swir2");
  var xigema = img.expression("(2 * blue + swir2 + red) / 4", {"blue": blue, "red": red, "swir2": swir2});
  var thresholdValue = ee.Image(0.2);
  var xigema_good = xigema.where(xigema.lt(thresholdValue), thresholdValue);

  var BR  = ((blue.subtract(red)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);
  var BS2 = ((blue.subtract(swir2)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);
  var RS2 = ((red.subtract(swir2)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);

  var kNDMI_1 = ee.Image(3)
    .subtract(ee.Image(2).multiply(img.expression('exp(b)', {'b': BR})))
    .subtract(ee.Image(2).multiply(img.expression('exp(b)', {'b': BS2})))
    .add(ee.Image(1).multiply(img.expression('exp(b)', {'b': RS2})));

  var kNDMI_2 = ee.Image(3)
    .add(ee.Image(2).multiply(img.expression('exp(b)', {'b': BR})))
    .add(ee.Image(2).multiply(img.expression('exp(b)', {'b': BS2})))
    .add(ee.Image(1).multiply(img.expression('exp(b)', {'b': RS2})));

  var kNDMI = ee.Image(1).subtract(kNDMI_1.divide(kNDMI_2));
  return kNDMI.rename("kNDMI");
}

function calkNDMI_005_fixed(img) {
  var blue = img.select("blue");
  var red = img.select("red");
  var swir2 = img.select("swir2");
  var xigema = img.expression(
    "(2 * blue + swir2 + red) / 4",
    {
      "blue": blue,
      "red": red,
      "swir2":swir2
    }
  );

  var xigema_good = ee.Image(0.05)

  var BR = ((blue.subtract(red)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);
  var k_BR = img.expression('exp(band)', {'band': BR});

  var BS2 = ((blue.subtract(swir2)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);
  var k_BS2 = img.expression('exp(band)', {'band': BS2});

  var RS2 = ((red.subtract(swir2)).pow(2)).divide(2).divide(xigema_good.pow(2)).multiply(-1);
  var k_RS2 = img.expression('exp(band)', {'band': RS2});

  var kNDMI_1 = ee.Image(3).subtract(ee.Image(2).multiply(k_BR)).subtract(ee.Image(2).multiply(k_BS2)).add(ee.Image(2).multiply(k_RS2));
  var kNDMI_2 = ee.Image(3).add(ee.Image(2).multiply(k_BR)).add(ee.Image(2).multiply(k_BS2)).add(ee.Image(2).multiply(k_RS2));

  var kNDMI_3 = kNDMI_1.divide(kNDMI_2)
  var kNDMI = ee.Image(1).subtract(kNDMI_3)
  return kNDMI.rename("kNDMI_005");
}

// ===============================
// 5) Monthly package: indices + QA
// ===============================
function computeMonthlyPackage(year, month) {
  var start = ee.Date.fromYMD(year, month, 1);
  var end   = start.advance(1, 'month');

  var baseCol = mod09a1.filterDate(start, end).map(prepareMod09);
  var hasData = baseCol.size().gt(0);

  // snow monthly summary
  var snowMonthly = monthlySnowSummary(start, end);
  var snowMaskNoSnow = snowMonthly.select('snowMaskNoSnow'); // 1 no-snow, 0 snow
  var snowFraction   = snowMonthly.select('snow_fraction').rename('snow_fraction');

  // Apply snow mask
  var baseSnow = baseCol.map(function(img){
    return img.updateMask(snowMaskNoSnow);
  });

  // n_valid_obs (after all masks)
  var nValidObs = ee.Image(ee.Algorithms.If(
    hasData,
    baseSnow.select('goodObs').sum(),
    ee.Image(0)
  )).rename('n_valid_obs').toUint16();

  // indices (monthly mean)
  var kSVMI_mean = ee.Image(ee.Algorithms.If(
    hasData,
    baseSnow.map(function(img){ return calkSVMI_005(img); }).mean(),
    ee.Image(0)
  )).rename('kSVMI');

  var kNDMI_020_mean = ee.Image(ee.Algorithms.If(
    hasData,
    baseSnow.map(function(img){ return calkNDMI_02_max(img); }).mean(),
    ee.Image(0)
  )).rename('kNDMI_020_max');

  var kNDMI_005_mean = ee.Image(ee.Algorithms.If(
    hasData,
    baseSnow.map(function(img){ return calkNDMI_005_fixed(img); }).mean(),
    ee.Image(0)
  )).rename('kNDMI_005_fixed');

  // valid_mask: no snow AND at least 1 valid 8-day obs
  var validMask = snowMaskNoSnow.eq(1).and(nValidObs.gte(1))
    .rename('valid_mask').toUint16();

  // qa_reason bitmask (UInt16), only for INVALID pixels; valid pixels = 0
  // bits: 1 noObs, 2 cloud, 4 shadow, 8 snow, 16 out-of-range
  var qaReason = ee.Image(0).rename('qa_reason').toUint16();

  var snowPresent = snowMaskNoSnow.eq(0);
  qaReason = qaReason.where(snowPresent, 8); // snow invalid

  // For non-snow invalid pixels: n_valid_obs==0
  var noObs = nValidObs.eq(0).and(snowPresent.not());
  qaReason = qaReason.where(noObs, qaReason.add(1));

  // Diagnose likely causes (any occurrence within the month)
  var rangeBadMonth  = ee.Image(ee.Algorithms.If(hasData, baseCol.select('rangeBad').max(), ee.Image(0))).eq(1);
  var cloudBadMonth  = ee.Image(ee.Algorithms.If(hasData, baseCol.select('cloudBad').max(), ee.Image(0))).eq(1);
  var shadowBadMonth = ee.Image(ee.Algorithms.If(hasData, baseCol.select('shadowBad').max(), ee.Image(0))).eq(1);

  qaReason = qaReason.where(noObs.and(rangeBadMonth),  qaReason.add(16));
  qaReason = qaReason.where(noObs.and(cloudBadMonth),  qaReason.add(2));
  qaReason = qaReason.where(noObs.and(shadowBadMonth), qaReason.add(4));

  // Mask indices by validMask (invalid pixels => NoData in INDEX export)
  var kSVMI_out       = kSVMI_mean.updateMask(validMask);
  var kNDMI_020_out   = kNDMI_020_mean.updateMask(validMask);
  var kNDMI_005_out   = kNDMI_005_mean.updateMask(validMask);

  return ee.Image.cat([
    kNDMI_005_out.rename('kNDMI_005_fixed'),
    kNDMI_020_out.rename('kNDMI_020_max'),
    kSVMI_out.rename('kSVMI'),
    snowFraction.rename('snow_fraction'),
    validMask.rename('valid_mask'),
    qaReason.rename('qa_reason'),
    nValidObs.rename('n_valid_obs')
  ]).set({
    'year': year,
    'month': month,
    'system:time_start': start.millis()
  });
}

// ===============================
// 6) Build monthly collection
// ===============================
function mm(m){ return (m < 10 ? '0' + m : '' + m); }

var months = ee.List.sequence(startMonth, endMonth);
var monthlyCollection = ee.ImageCollection(
  months.map(function(m){ return computeMonthlyPackage(year, ee.Number(m).int()); })
);

// Optional preview
var april = monthlyCollection.filter(ee.Filter.eq('month', 4)).first();
Map.addLayer(april.select('kSVMI'), {min:0, max:1}, 'kSVMI (Apr)');
Map.addLayer(april.select('kNDMI_020_max'), {min:0, max:1}, 'kNDMI_020_max (Apr)');
Map.addLayer(april.select('kNDMI_005_fixed'), {min:0, max:1}, 'kNDMI_005_fixed (Apr)');
Map.addLayer(april.select('snow_fraction'), {min:0, max:1}, 'snow_fraction (Apr)');

// ===============================
// 7) Export (Scheme A): two multiband files per month
// ===============================
for (var m = startMonth; m <= endMonth; m++) {
  var img = monthlyCollection.filter(ee.Filter.eq('month', m)).first();
  var ym = year + '_' + mm(m);

  // INDEX package (Float32) - explicit NoData
  Export.image.toDrive({
    image: img.select(['kNDMI_005_fixed','kNDMI_020_max','kSVMI','snow_fraction'])
              .toFloat()
              .unmask(-9999),
    description: 'INDEX_' + ym,
    folder: 'kNDMI_kSVMI_INDEX',
    fileNamePrefix: 'INDEX_' + ym,
    region: exportRegion,
    scale: exportScale,
    crs: exportCRS,
    fileFormat: 'GeoTIFF',
    formatOptions: {
      cloudOptimized: true,
      noData: -9999
    },
    maxPixels: 1e13
  });

  // QA package (UInt16) - no noData; use valid_mask
  Export.image.toDrive({
    image: img.select(['valid_mask','qa_reason','n_valid_obs'])
              .toUint16()
              .unmask(0),
    description: 'QA_' + ym,
    folder: 'kNDMI_kSVMI_QA',
    fileNamePrefix: 'QA_' + ym,
    region: exportRegion,
    scale: exportScale,
    crs: exportCRS,
    fileFormat: 'GeoTIFF',
    formatOptions: {
      cloudOptimized: true
    },
    maxPixels: 1e13
  });
}
