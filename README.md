## Joy Devistion plot from dem with bound of dem

- directly from [here](https://hannes.enjoys.it/blog/2019/09/dynamic-elevation-profile-lines-as-qgis-geometry-generator/) only restricted to the bounds of the raster

```
-- UPPER CASE comments below are where you can change things

with_variable(
	'raster_layer',
	'dem',  -- RASTER LAYER to sample from
	-- this collects all the linestrings generated below into one multilinestring
	collect_geometries(
		-- a loop for each y value of the grid
		array_foreach(
			-- array_foreach loops over all elements of the series generated below
			-- which is a range of numbers from the bottom to the top of y values
			-- of the map canvas extent coordinates.
			-- the result will be an array of linestrings
			generate_series(
                y_min(layer_property(@raster_layer, 'extent')),
                y_max(layer_property(@raster_layer, 'extent')),
                bounds_height(layer_property(@raster_layer, 'extent')) / 50
			),
			
			-- we want to enter another loop so we assign the name 'y' to
			-- the current element of the array_foreach loop
			with_variable(
				'y',
				@element,
				
				-- now we are ready to generate the line for this y value
				make_line(
					-- another loop, this time for the x values. same logic as before
					-- the result will be an array of points
					array_foreach(
						generate_series(
                        x_min(layer_property(@raster_layer, 'extent')),
                        x_max(layer_property(@raster_layer, 'extent')),
                        bounds_width(layer_property(@raster_layer, 'extent')) / 50
						),
						-- and here we create each point of the line
						make_point(
							@element,  -- the current value from the loop over the x value range
							@y  -- the y value from the outer loop
							+   -- will get an additional offset to generate the effect
							-- we look for values at _this point_ in the raster, and since
							-- the raster might not have any value here, we must use coalesce
							-- to use a replacement value in those cases
							coalesce(  -- coalesce to catch raster null values
								raster_value(
									@raster_layer,
									1,  -- band 1, *snore*
									-- to look up the raster value we need to look in the right position
									-- so we make a sampling point in the same CRS as the raster layer
									transform( 
										make_point(@element, @y),
										@map_crs,
										layer_property(@raster_layer,'crs')
									)
								),
								0  -- coalesce 0 if raster_value gave null
							-- here is where we set the scaling factor for the raster -> y values
							-- if things are weird, set it to 0 and try small multiplications or divisions
							-- to see what happens.
							-- for metric systems you will want to multiply
							-- for geographic coordinates you will want to divide
							)*20  -- user-defined factor for VERTICAL EXAGGERATION
						)
					)
				)
			)
		)
	)
)  -- wee
```