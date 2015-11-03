# Deep indent

Deep indentation level is a signal of an unreadable code.

```js
function retrieveSpecsForBasicFleet(savedModel) {
  return DisplayedSpecification
    .getByModelRdbIdWithProductName(savedModel.model_rdb_id, dataSetToProduct('costs'))
    .then(function (displayedSpecs) {
      return Configuration.findByConfigurationId(savedModel.configuration_id)
        .then(function (confs) {
          var configuration = confs.specification
            .map(function (specsForConf) {
              return specsForConf.specs.filter(function (specsForConfItem) {
                return displayedSpecs.some(function (displayedSpecsItem) {
                  return displayedSpecsItem.specification_name === specsForConfItem.name;
                });
              });
            })
            .reduce(function (prev, curr) {
              return prev.concat(curr);
            });

          return {
            id: savedModel.unique_id,
            subcategory: savedModel.subtype_name,
            manufacturer: savedModel.manufacturer_name,
            model: savedModel.model_name,
            year: savedModel.model_year,
            serialNumber: savedModel.serial_number,
            configuration: configuration,
            notes: savedModel.notes
          };
        });
    });
}
```

Deepest point: 18 columns.

# Refactoring process

There is nothing else to do with unreadable code then refactor it, i.e. apply changes to it step by step without
changing the way it works.

## Name every function

First convert each functional expression into function declaration, this is possible only by giving names to all
your functions. Start from deepest one, and remember to preserve the same scope, so there is no need to carefully
check that all needed variables are passed as parameters (we will do this later).

Think about names. Put everything function does in it's name. Function name gets too long? Great, it is a sign of
a needed refactoring, we will do it later.

```js
function retrieveSpecsForBasicFleet(savedModel) {
  return DisplayedSpecification
    .getByModelRdbIdWithProductName(savedModel.model_rdb_id, dataSetToProduct('costs'))
    .then(findConfigurationAndChangeTheWorld);
    
  // FIXME this function does way more than just finding configuration
  function findConfigurationAndChangeTheWorld(displayedSpecs) {
    return Configuration.findByConfigurationId(savedModel.configuration_id)
      .then(toReportRecord);
      
    // Note: from external code we seen that `retrieveSpecsForBasicFleet`
    // resolves to a report record.
    function toReportRecord(confs) {
      var configuration = confs.specification
        .map(someMap)
        .reduce(concat);
        
      // TODO give a better name
      function someMap(specsForConf) {
        return specsForConf.specs.filter(unclearFilter);
        
        // TODO give a better name
        function unclearFilter(specsForConfItem) {
          return displayedSpecs.some(sameSpecName);
          
          function sameSpecName(displayedSpecsItem) {
            return displayedSpecsItem.specification_name === specsForConfItem.name;
          }
        }
      }
        
      function concat(prev, curr) {
        return prev.concat(curr);
      }

      return {
        id: savedModel.unique_id,
        subcategory: savedModel.subtype_name,
        manufacturer: savedModel.manufacturer_name,
        model: savedModel.model_name,
        year: savedModel.model_year,
        serialNumber: savedModel.serial_number,
        configuration: configuration,
        notes: savedModel.notes
      };
    }
  }
}
```

Deepest point: 12 columns (two thirds from original depth).

Now code is less nested, which is great, but look at those functions: `findConfigurationAndChangeTheWorld`, `someMap`, `unclearFilter`... they received these names because they do exactly this, _something unclear_.

## Prepare data and use lodash

Try to use lodash every time you see boilerplate code working with arrays or collections.

Lets look at `toReportRecord`. What it does?

```js
// Note: `displayedSpecs` are coming from outer scope.

function toReportRecord(confs) {
  var configuration = confs.specification
    .map(someMap)
    .reduce(concat);
    
  // TODO give a better name
  function someMap(specsForConf) {
    return specsForConf.specs.filter(unclearFilter);
    
    // TODO give a better name
    function unclearFilter(specsForConfItem) {
      return displayedSpecs.some(sameSpecName);
      
      function sameSpecName(displayedSpecsItem) {
        return displayedSpecsItem.specification_name === specsForConfItem.name;
      }
    }
  }
    
  function concat(prev, curr) {
    return prev.concat(curr);
  }

  // return ... 
}
```

Lets first clearly define what we do here and on which data we operate, based on exising code we can see the data
structures used:

```js
var displayedSpecs = [
  {
    specification_name: String
  }
];

var confs = {
  specification: [
    {
      specs: [
        {
          name: String // maps to specification_name
        }
      ]
    }
  ]
}
```

The code essentially extracts all items from `specs` which names are seen in `displayedSpecs`. We strive to see
clear code which reads the same as previous sentence. Easy, but why we have so much code? It is because data is
unprepared.

Lets cook the data!

```js
// Note: `displayedSpecs` are coming from outer scope.

function toReportRecord(confs) {
  var displayedNames = _.map(displayedSpecs, 'specification_name');
  var specs = _(confs.specification).map('specs').flatten().value();
  
  // FIXME now we see that `configuration` is a bad name for this variable
  var configuration = _.filter(specs, function (spec) {
    return _.includes(displayedNames, spec.name);
  });

  // return ... 
}
```

Result after this step:

```js
function retrieveSpecsForBasicFleet(savedModel) {
  return DisplayedSpecification
    .getByModelRdbIdWithProductName(savedModel.model_rdb_id, dataSetToProduct('costs'))
    .then(findConfigurationAndChangeTheWorld);

  // FIXME this function does way more than just finding configuration
  function findConfigurationAndChangeTheWorld(displayedSpecs) {
    return Configuration.findByConfigurationId(savedModel.configuration_id)
      .then(toReportRecord);

    function toReportRecord(confs) {
      var displayedNames = _.map(displayedSpecs, 'specification_name');
      var specs = _(confs.specification).map('specs').flatten().value();
      
      // FIXME now we see that `configuration` is a bad name for this variable
      var configuration = _.filter(specs, function (spec) {
        return _.includes(displayedNames, spec.name);
      });

      return {
        id: savedModel.unique_id,
        subcategory: savedModel.subtype_name,
        manufacturer: savedModel.manufacturer_name,
        model: savedModel.model_name,
        year: savedModel.model_year,
        serialNumber: savedModel.serial_number,
        configuration: configuration,
        notes: savedModel.notes
      };
    }
  }
}
```

Deepest point: 8 columns (remember that original depth was 18).

## Pass every variable as a parameter and extract nested functions

Lets extract _innermost_ function first, `toReportRecord`.

```js
function retrieveSpecsForBasicFleet(savedModel) {
  return DisplayedSpecification
    .getByModelRdbIdWithProductName(savedModel.model_rdb_id, dataSetToProduct('costs'))
    .then(findConfigurationAndChangeTheWorld);

  // FIXME this function does way more than just finding configuration
  function findConfigurationAndChangeTheWorld(displayedSpecs) {
    return Configuration.findByConfigurationId(savedModel.configuration_id)
      .then(function (confs) {
        return toReportRecord(savedModel, displayedSpecs, confs);
      });
  }
}

function toReportRecord(savedModel, displayedSpecs, confs) {
  var displayedNames = _.map(displayedSpecs, 'specification_name');
  var specs = _(confs.specification).map('specs').flatten().value();
  
  // FIXME now we see that `configuration` is a bad name for this variable
  var configuration = _.filter(specs, function (spec) {
    return _.includes(displayedNames, spec.name);
  });

  return {
    id: savedModel.unique_id,
    subcategory: savedModel.subtype_name,
    manufacturer: savedModel.manufacturer_name,
    model: savedModel.model_name,
    year: savedModel.model_year,
    serialNumber: savedModel.serial_number,
    configuration: configuration,
    notes: savedModel.notes
  };
}
```

Now we can see that `retrieveSpecsForBasicFleet` is quite simple and we can notice that promises are nested for
no reason: `findConfigurationAndChangeTheWorld` does not need `displayedSpecs` until promise of
`findByConfigurationId` is resolved. Lets flatten them!

## Flatten promises

Here is what happens from your flatten promises:

```js
function retrieveSpecsForBasicFleet(savedModel) {
  var displayedSpecs = DisplayedSpecification.getByModelRdbIdWithProductName(
    savedModel.model_rdb_id,
    dataSetToProduct('costs')
  );
  var configuration = Configuration.findByConfigurationId(savedModel.configuration_id);
  
  // FIXME we return report record, but function says that it retrieves specs
  return P.all([savedModel, displayedSpecs, configuration]).spread(toReportRecord);
}

// FIXME `confs` is a weird name for `configuration`
function toReportRecord(savedModel, displayedSpecs, confs) {
  var displayedNames = _.map(displayedSpecs, 'specification_name');
  var specs = _(confs.specification).map('specs').flatten().value();
  
  // FIXME now we see that `configuration` is a bad name for this variable
  var configuration = _.filter(specs, function (spec) {
    return _.includes(displayedNames, spec.name);
  });

  return {
    id: savedModel.unique_id,
    subcategory: savedModel.subtype_name,
    manufacturer: savedModel.manufacturer_name,
    model: savedModel.model_name,
    year: savedModel.model_year,
    serialNumber: savedModel.serial_number,
    configuration: configuration,
    notes: savedModel.notes
  };
}
```

Deepest point: 4 columns (original depth was 18).

That's all. All other things should be easily fixable using common sense.

P.S. Refactoring made `toReportRecord` a pure function which is great for unit testing. There was no unit tests for initial code, good luck trying to write them without mocking the universe.

P.P.S. Refactor your code until you could unit test it. If you could not test it, nobody can read it.
