<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { extent } from "d3-array";

	let instances = [];
	let xScale;
	let yScale;

	const numClasses = 10;
	const numBins = 10;
	let binsByClasses = [];

	const scatterHeight = 500;
	const scatterWidth = 450;
	const buffer = 10;

	let colors = ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c','#fdbf6f','#ff7f00','#cab2d6','#6a3d9a']
	let colorScale = () => {};

	let bin_scale;
	let bin_ticks = [];

	let selected_id;
	let selected_filename;
	let selected_true_label;
	let selected_predicted_label;
	let image_path = './static/images/';

	let clickable_images = 0;
	let clickable_class_text = 0;
	let selected_ids = new Set();
	let selected_class_flag = false;

	onMount(async () => {
		const fetched = await fetch("./static/prediction_results.json");
		instances = (await fetched.json()).test_instances;
		console.log(instances);

		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}
		console.log(binsByClasses);

		// console.log(binsByClasses[0]);
		// binsByClasses[0]['instances'].push(3);
		// console.log(binsByClasses[0].instances);
		// binsByClasses[0].instances.push(3);
		// console.log(binsByClasses[0].bins[3]);
		
		instances.forEach(instance => {
			let true_class = instance.true_label;
			let bin_value = instance.predicted_scores[true_class];
			
			if (bin_value >= 0 && bin_value < 0.1) {
				binsByClasses[true_class].bins[0].instances.push(instance.id);
			}
			else if (bin_value >= 0.1 && bin_value < 0.2) {
				binsByClasses[true_class].bins[1].instances.push(instance.id);
			}
			else if (bin_value >= 0.2 && bin_value < 0.3) {
				binsByClasses[true_class].bins[2].instances.push(instance.id);
			}
			else if (bin_value >= 0.3 && bin_value < 0.4) {
				binsByClasses[true_class].bins[3].instances.push(instance.id);
			}
			else if (bin_value >= 0.4 && bin_value < 0.5) {
				binsByClasses[true_class].bins[4].instances.push(instance.id);
			}
			else if (bin_value >= 0.5 && bin_value < 0.6) {
				binsByClasses[true_class].bins[5].instances.push(instance.id);
			}
			else if (bin_value >= 0.6 && bin_value < 0.7) {
				binsByClasses[true_class].bins[6].instances.push(instance.id);
			}
			else if (bin_value >= 0.7 && bin_value < 0.8) {
				binsByClasses[true_class].bins[7].instances.push(instance.id);
			}
			else if (bin_value >= 0.8 && bin_value < 0.9) {
				binsByClasses[true_class].bins[8].instances.push(instance.id);
			}
			else {
				binsByClasses[true_class].bins[9].instances.push(instance.id);
			}
		});
		
		console.log('After Binning: ', binsByClasses);

		/*
		binsByClasses.forEach(bin => {
			let class_bin = [];
			for (var i=0;i<10;i++) {
				let current_bin = [];
				bin.bins[i].instances.forEach(id => {
					current_bin.push(id);
				});
				class_bin.push(current_bin);
				// console.log(class_bin);
			}
			true_class_ids.push(class_bin);
		});
		// console.log('Here', true_class_ids);
		*/

		let projection_x = [];
		let projection_y = [];

		instances.forEach(instance => {
			projection_x.push(instance.projection[0]);
			projection_y.push(instance.projection[1]);

		});
		// console.log(projection_x);

		let xExtent = extent(projection_x);
		let yExtent = extent(projection_y);
		
		xScale = scaleLinear().domain(xExtent).range([buffer, scatterWidth - (buffer*4)]);
		yScale = scaleLinear().domain(yExtent).range([scatterHeight - buffer, buffer]);

		let classes = new Set(instances.map((d) => d.true_label));
		// console.log(classes);

		colorScale = scaleOrdinal().domain(Array.from(classes)).range(colors);

		bin_scale = scaleLinear().domain([0,1]).range([0,9]);
		bin_ticks = bin_scale.ticks(10);
		// console.log(bin_ticks.length, bin_ticks);

		let image_x = 15;
		let image_y = 60;
		let image_stack_y = 0;

		// CLASS 0 Bins populating
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x);
			svg.setAttribute('y', image_y);
			if (binsByClasses[0].bins[i].instances.length <= 0) {
				console.log('Empty');
				image_x = image_x + 75;
				//image_x = image_x + 80;
			}
			binsByClasses[0].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);

				if (binsByClasses[0].bins[i].instances.length > 6) {
					svg.setAttribute('y', image_y - 40);		// Dynamically suggest subtraction based on number of entries in instacne.
				}

				if ((j/6) >= 1) {
					image_stack_y = 10 * Math.floor(j/6);
					j = (j%6);
					
				}
				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', 15*j);
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_x = image_x + 33;
		};

		image_x = 15;
		image_y = 115;
		image_stack_y = 0;

		// CLASS 1 Bins populating
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x);
			svg.setAttribute('y', image_y);
			if (binsByClasses[1].bins[i].instances.length <= 0) {
				console.log('Empty');
				image_x = image_x + 30;
				//image_x = image_x + 80;
			}
			binsByClasses[1].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[1].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[1].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*7));		// Dynamically suggest subtraction based on number of entries in instacne.
				}

				if ((j/6) >= 1) {
					image_stack_y = 10 * Math.floor(j/6);
					j = (j%6);
					
				}
				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', 15*j);
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 11;

			});
			image_x = image_x + 57;
		};

		image_x = 18;
		image_y = 190;
		image_stack_y = 0;

		// CLASS 2 Bins populating
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);
			let x_of_bin = image_x;
			let y_of_bin = image_y;

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x);
			svg.setAttribute('y', image_y);
			if (binsByClasses[2].bins[i].instances.length <= 0) {
				console.log('Empty');
				image_x = image_x + 60;
				//image_x = image_x + 80;
			}
			binsByClasses[2].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[2].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[2].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*7)-10);		// Dynamically suggest subtraction based on number of entries in instacne.
				}

				let buff = 0;
				if (binsByClasses[2].bins[i].instances.length == 5) {
					buff = 25;
				}
				if (binsByClasses[2].bins[i].instances.length == 2) {
					buff = 15;
				}
				if (binsByClasses[2].bins[i].instances.length == 1) {
					buff = 15;
				}

				if ((j/6) >= 1) {
					image_stack_y = 10 * Math.floor(j/6);
					j = (j%6);
					image_x = image_x - 10;
				}
				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j)+buff);
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 12;

			});
			image_x = image_x + 44;
		};

		image_x = 18;
		image_y = 250;
		image_stack_y = 0;

		// CLASS 3 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);
			
			let buff = 0;
			let buff2 = 0;
			if (binsByClasses[3].bins[i].binNo == 3) {
				buff = -10;
			}
			if (binsByClasses[3].bins[i].binNo == 6) {
				buff = 110;
			}
			if (binsByClasses[3].bins[i].binNo == 8) {
				buff = 200;
			}
			if (binsByClasses[3].bins[i].binNo == 9) {
				buff = 250;
				buff2 = -10;
			}

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', (image_y+buff2));
			if (binsByClasses[3].bins[i].instances.length <= 0) {
				console.log('Empty');
				image_x = image_x + 60;
				//image_x = image_x + 80;
			}
			binsByClasses[3].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[3].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[3].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8)+buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			//image_x = image_x + 44;
		};
		
		image_x = 18;
		image_y = 330;
		image_stack_y = 0;

		// CLASS 4 Bins populating
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x);
			svg.setAttribute('y', image_y);
			if (binsByClasses[4].bins[i].instances.length <= 0) {
				console.log('Empty');
				image_x = image_x + 75;
				//image_x = image_x + 80;
			}
			binsByClasses[4].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[4].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[4].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*7)-10);		// Dynamically suggest subtraction based on number of entries in instacne.
				}

				
				if (binsByClasses[4].bins[i].binNo == 0) {
					image_stack_y = 15;
				}

				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			image_x = image_x + 10;
		};

		image_x = 10;
		image_y = 410;
		image_stack_y = 0;

		// CLASS 5 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			let buff = 0;
			let buff2 = 0;
			if (binsByClasses[5].bins[i].binNo == 2) {
				buff = -30;
			}
			if (binsByClasses[5].bins[i].binNo == 4) {
				buff = 20;
			}
			if (binsByClasses[5].bins[i].binNo == 8) {
				buff = 85;
			}
			if (binsByClasses[5].bins[i].binNo == 9) {
				buff = 89;
				buff2 = -10;
			}
			
			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', image_y+buff2);
			if (binsByClasses[5].bins[i].instances.length <= 0) {
				//console.log('Empty');
				image_x = image_x + 85;
				//image_x = image_x + 80;
			}


			binsByClasses[5].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[5].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[5].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8) + buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			//image_x = image_x + 10;
		};

		
		image_x = 10;
		image_y = 475;
		image_stack_y = 0;

		// CLASS 6 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			let buff = 0;
			let buff2 = 0;
			if (binsByClasses[6].bins[i].binNo == 6) {
				buff = -30;
			}
			if (binsByClasses[6].bins[i].binNo == 7) {
				buff = -92;
			}
			if (binsByClasses[6].bins[i].binNo == 8) {
				buff = -170;
			}
			if (binsByClasses[6].bins[i].binNo == 9) {
				buff = -250;
				buff2 = -5;
			}

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', image_y+buff2);
			if (binsByClasses[6].bins[i].instances.length <= 0) {
				//console.log('Empty');
				image_x = image_x + 10;
				//image_x = image_x + 80;
			}


			binsByClasses[6].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[6].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[6].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8) + buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			image_x = image_x + 72;
		};

		image_x = 10;
		image_y = 540;
		image_stack_y = 0;

		// CLASS 7 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			let buff = 0;
			let buff2 = 0;
			
			if (binsByClasses[7].bins[i].binNo == 7) {
				buff = 52;
			}
			if (binsByClasses[7].bins[i].binNo == 8) {
				buff = 60;
			}
			if (binsByClasses[7].bins[i].binNo == 9) {
				buff = -127;
				buff2 = -8;
			}

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', image_y+buff2);
			if (binsByClasses[7].bins[i].instances.length <= 0) {
				//console.log('Empty');
				image_x = image_x + 95;
				//image_x = image_x + 80;
			}


			binsByClasses[7].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[7].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[7].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8) + buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			//image_x = image_x + 72;
		};

		image_x = 10;
		image_y = 605;
		image_stack_y = 0;

		// CLASS 8 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			let buff = 0;
			let buff2 = 0;
			
			if (binsByClasses[8].bins[i].binNo == 1) {
				buff = -35;
			}
			if (binsByClasses[8].bins[i].binNo == 2) {
				buff = 2;
			}
			if (binsByClasses[8].bins[i].binNo == 3) {
				buff = -17;
			}
			if (binsByClasses[8].bins[i].binNo == 4) {
				buff = 5;
			}
			if (binsByClasses[8].bins[i].binNo == 5) {
				buff = 30;
			}
			if (binsByClasses[8].bins[i].binNo == 6) {
				buff = 40;
			}
			if (binsByClasses[8].bins[i].binNo == 7) {
				buff = 80;
			}
			if (binsByClasses[8].bins[i].binNo == 8) {
				buff = 100;
			}

			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', image_y+buff2);
			if (binsByClasses[8].bins[i].instances.length <= 0) {
				//console.log('Empty');
				image_x = image_x + 20;
				//image_x = image_x + 80;
			}


			binsByClasses[8].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[8].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[8].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8) + buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			//image_x = image_x + 50;
		};

		image_x = 10;
		image_y = 670;
		image_stack_y = 0;

		// CLASS 9 Bins populating
		// Well thoughtout Variables
		for (var i=0;i<numBins;i++) {

			// console.log(bin.bins[i].instances);

			let buff = 0;
			let buff2 = 0;
			
			if (binsByClasses[9].bins[i].binNo == 3) {
				buff = 20;
			}
			if (binsByClasses[9].bins[i].binNo == 4) {
				buff = 30;
			}
			if (binsByClasses[9].bins[i].binNo == 5) {
				buff = 70;
			}
			if (binsByClasses[9].bins[i].binNo == 6) {
				buff = 75;
			}
			if (binsByClasses[9].bins[i].binNo == 8) {
				buff = 95;
			}
			if (binsByClasses[9].bins[i].binNo == 9) {
				buff = 135;
				buff2 = -10;
			}


			var svg = document.createElementNS('http://www.w3.org/2000/svg','svg');
			svg.setAttribute('xmlns:xlink','http://www.w3.org/1999/xlink');
			svg.setAttribute('height','100');
			svg.setAttribute('width','100%');
			svg.setAttribute('id','image-window');
			//svg.setAttribute('preserveAspectRatio', "xMaxYMax meet");
			svg.setAttribute('x', image_x+buff);
			svg.setAttribute('y', image_y+buff2);
			if (binsByClasses[9].bins[i].instances.length <= 0) {
				//console.log('Empty');
				image_x = image_x + 55;
				//image_x = image_x + 80;
			}


			binsByClasses[9].bins[i].instances.forEach(function (id, j) {
				
				let image_address = '/' + image_path + 'image-' + id + '.png';
				// console.log('Here', id, image_address);
				let number_lines_in_bins;

				if (binsByClasses[9].bins[i].instances.length > 6) {
					number_lines_in_bins = Math.floor(binsByClasses[9].bins[i].instances.length / 6);
					svg.setAttribute('y', image_y - (number_lines_in_bins*8) + buff2);		// Dynamically suggest subtraction based on number of entries in instacne.
				}


				if ((j/6) >= 1) {
					image_stack_y = 11 * Math.floor(j/6);
					j = (j%6);
					//image_x = image_x - 10;
				}

				var svgimg = document.createElementNS('http://www.w3.org/2000/svg','image');
				svgimg.setAttribute('class','image-svg');
				svgimg.setAttribute('id', 'image-'+id);
				svgimg.setAttributeNS('http://www.w3.org/1999/xlink','href', image_address);
				svgimg.setAttribute('x', (15*j));
				svgimg.setAttribute('y', image_stack_y);
				svgimg.setAttribute('height','8');
				svgimg.setAttribute('width','8');
				
				
				svg.appendChild(svgimg);
				
				document.querySelector('#images-svg').appendChild(svg);
				//document.querySelector('#images-svg').appendChild(p_temp);

				image_x = image_x + 15;

			});
			image_stack_y = 0;
			image_x = image_x + 30;
		};

		// Free Up Memory
		binsByClasses = null;
		console.log(binsByClasses);

		clickable_images = document.getElementsByClassName('image-svg');
		//console.log(clickable_images);

		for (var i=0; i < clickable_images.length; i++) {
			clickable_images[i].onclick = function() {
				console.log(this.id, this.href);
				selected_id = this.id.substring(this.id.indexOf('-')+1);
				selected_filename = this.href.baseVal.substring(this.href.baseVal.indexOf('-')-5);
				console.log(selected_id, selected_filename);
				selected_predicted_label = instances[selected_id].predicted_label;
				selected_true_label = instances[selected_id].true_label;

				let selected_point = document.getElementsByClassName('point', 'seleced');
				selected_point.opacity = 1;
			}
		};

		
		clickable_class_text = document.querySelectorAll(".class-text");
		console.log('This', clickable_class_text);
		for(var i=0; i < clickable_class_text.length; i++) {
			clickable_class_text[i].onclick = function() {
				let text = this.textContent;
				text = text.substring(text.indexOf(' ')+1);
				//console.log(text);

				if (selected_class_flag) {
					selected_ids.clear();
				}

				instances.forEach(instance => {
					if (text == instance.true_label) {
						selected_ids.add(instance.id);
						selected_class_flag = true;
					}
				});
			}
			console.log(selected_ids);	
		}
		console.log(selected_ids);
	});


</script>

<main>

	<h1>Visual Analytics HW 3</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<svg class='scatter-view' height={scatterHeight} width={scatterWidth}>
					{#each instances as instance}
						<circle
							id= "datapoint-{instance.id} {selected_id == instance.id ? 'selected-point' : ''}"
							class= "point {selected_id == instance.id ? 'selected' : ''}" 
							r= {selected_id != instance.id? 3 : 5} 
							cx={xScale(instance.projection[0])}
							cy={yScale(instance.projection[1])} 
							fill={colorScale(instance.true_label)} 
							stroke={colorScale(instance.predicted_label)} 
							stroke-width= {selected_id != instance.id? 2 : 3.5}
							opacity= {selected_id == instance.id || selected_id == null? 0.9 : 0.15}
							on:click = {() => {
								selected_id = instance.id;
								selected_filename = instance.filename;
								selected_true_label = instance.true_label;
								selected_predicted_label = instance.predicted_label;
								{console.log(selected_filename)}
								// selectedPoint();
							}}
							/>
					{/each}

				</svg>
			</div>

			<div id="selected-image-view" class="view-panel">
				<div class="view-title">Selected Image</div>
				<div id="selected-image-view-content" class="flex-container">
					{#if selected_id != undefined}
						<div id="selected-image" class="flex-child"> 
							<img src={image_path + selected_filename} alt=""/>
						</div>
						<div id="selected-image-desc" class="flex-child"> 
							<p>Image Id: {selected_id}</p>
							<p>True Label: {selected_true_label}</p>
							<p>Predicted Label: {selected_predicted_label}</p>
						</div>
					{/if}
				</div>
			</div>
		</div>
		
		<div id="main-section" style="width: 1000px;">
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Score Distributions</div>
				<div class="flex-container">
					<div id="labels-axis" class="flex-child">
						<svg width=100 height=685>

							<text x="3" y="45" class="class-text">Class 0</text>
							<text x="3" y="58" class="labelled-text">Labelled as 0</text>
							<text x="3" y="70" class="predicted-text">Predicted as 0</text>
							
							<text x="3" y="110" class="class-text">Class 1</text>
							<text x="3" y="123" class="labelled-text">Labelled as 1</text>
							<text x="3" y="135" class="predicted-text">Predicted as 1</text>
							
							<text x="3" y="175" class="class-text">Class 2</text>
							<text x="3" y="188" class="labelled-text">Labelled as 2</text>
							<text x="3" y="200" class="predicted-text">Predicted as 2</text>
							
							<text x="3" y="240" class="class-text">Class 3</text>
							<text x="3" y="253" class="labelled-text">Labelled as 3</text>
							<text x="3" y="265" class="predicted-text">Predicted as 3</text>

							<text x="3" y="330" class="class-text">Class 4</text>
							<text x="3" y="343" class="labelled-text">Labelled as 4</text>
							<text x="3" y="355" class="predicted-text">Predicted as 4</text>

							<text x="3" y="395" class="class-text">Class 5</text>
							<text x="3" y="408" class="labelled-text">Labelled as 5</text>
							<text x="3" y="420" class="predicted-text">Predicted as 5</text>

							<text x="3" y="460" class="class-text">Class 6</text>
							<text x="3" y="473" class="labelled-text">Labelled as 6</text>
							<text x="3" y="485" class="predicted-text">Predicted as 6</text>

							<text x="3" y="500" class="class-text">Class 7</text>
							<text x="3" y="513" class="labelled-text">Labelled as 7</text>
							<text x="3" y="525" class="predicted-text">Predicted as 7</text>
							
							<text x="3" y="575" class="class-text">Class 8</text>
							<text x="3" y="588" class="labelled-text">Labelled as 8</text>
							<text x="3" y="600" class="predicted-text">Predicted as 8</text>

							<text x="3" y="640" class="class-text">Class 9</text>
							<text x="3" y="653" class="labelled-text">Labelled as 9</text>
							<text x="3" y="665" class="predicted-text">Predicted as 9</text>
						</svg>
					</div>
					<div id="img-bins" class="flex-child">
						<svg id="images-svg" width=850 height=685>

							<line x1="10" x2="848" y1="15" y2="15" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="12" y2="5" stroke="maroon"/>
										<text x="-5" y="3" fill="maroon">{tick}</text>
									</g>
								{/each}
							</g>

							<line x1="10" x2="848" y1="70" y2="70" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="53" y2="60" stroke="maroon"/>
									</g>
								{/each}
							</g>
						

							<line x1="10" x2="848" y1="135" y2="135" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="118" y2="125" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<line x1="10" x2="848" y1="200" y2="200" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="183" y2="190" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<line x1="10" x2="848" y1="265" y2="265" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="248" y2="255" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<line x1="10" x2="848" y1="355" y2="355" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="338" y2="345" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<!--Class 5 Axis Line-->
							<line x1="10" x2="848" y1="420" y2="420" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="403" y2="410" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<!--Class 6 Axis Line-->
							<line x1="10" x2="848" y1="485" y2="485" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="468" y2="475" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<!--Class 7 Axis Line-->
							<line x1="10" x2="848" y1="550" y2="550" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="533" y2="540" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<!--Class 8 Axis Line-->
							<line x1="10" x2="848" y1="615" y2="615" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="598" y2="605" stroke="maroon"/>
									</g>
								{/each}
							</g>

							<!--Class 9 Axis Line-->
							<line x1="10" x2="848" y1="680" y2="680" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="663" y2="670" stroke="maroon"/>
									</g>
								{/each}
							</g>
								
						</svg>
						<svg width=100% height=35>
						<line x1="10" x2="848" y1="20" y2="20" stroke="maroon"/>
							<g id="axis-ticks" transform="translate(10, 0)">
								{#each bin_ticks as tick}
									<g transform="translate({bin_scale(tick)*93}, 10)">
										<line class="axis" y1="3" y2="10" stroke="maroon"/>
										<text x="-5" y="25" fill="maroon">{tick}</text>
									</g>
								{/each}
							</g>
						</svg>
					</div>
				</div>

			</div>
		</div>
	</div>
</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
	}
	#selected-image-view-content {
		height: 100px;
		padding: 10px;
	}
	.flex-container {
		display: flex;
	}
	#selected-image {
		flex: 0.3;
	}
	#selected-image-desc {
		flex: 0.7;
		background-color: #f3f3f3;
		border: 2px solid #eee;
		font-size: 0.8rem;
		padding-left: 5px;
	}
	#selected-image img{
		height: 80px;
		width: 80px;
	}
	svg {
		margin: 5px;
	}
	#labels-axis {
		background-color: #f3f3f3;
		border: 2px solid rgb(92, 92, 112);
	}
	.labelled-text {
		font-size: 0.8rem;
	}
	.predicted-text {
		font-size: 0.8rem;
	}
	#img-bins {
		flex: 1;
		background-color: none;
	}
	#image-window {
		display: flex;
	}
</style>
