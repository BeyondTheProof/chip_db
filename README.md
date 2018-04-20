# chip_db
A python implementation of a lightweight database to deal with many ChIP-seq or similar whole-genome tracks. Allows for constant-time access of any position along the genome.

You will first need to create the database using the provided python script. Use it as follows:

 	python chip_db.py cell_type data_resolution genome_size.txt data_files

Argument:
cell_type: It can be whatever you want. It doesn't need to match any filenames or anything.
data_resolution: I normally use 50bp, but if you want to try to go for finer resolution, go ahead! Just remember that the finer the resolution, the bigger the file and the longer it takes to access a genomic region.
genome: this should be a filepath to a chromosome size file and have a format like this: http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.chrom.sizes. It's simply a tab-delimited text file with the chromosome name in the first field and length (in bp) in the second field.
data: This can be any number of arguments. This is the data that will be added to your database. The files can be either in bedgraph format or fixed-step wig format.

The creation of the database will take a while. If there are any errors, you will most likely need to restart, so you can either do it in batches and make checkpoints or make sure you can do it all in one run.

Now for usage. In your python code, you will want to have the following:

	from chip_db import chip_data
	hdfile = h5py.File(database_filename.hdf5, 'r')
	grp = hdfile['/'+cell_type+'/'+data_resolution]
	dset = grp[u'chip_tracks']
	curr_db = chip_data(hdfile, grp, dset, cell_type, tracks=tracks)

To get data from a certain region of the genome, do:

	curr_db.get_data(chromosome, start_pos, end_pos, res=None, bin_method='mean', transform=None)

This will return an np array of size (num_tracks, length). Here's some more about the arguments:
chromosome: A string with the same name as you had in your chromosome size / data files
start_pos / end_pos: Integers corresponding to the start and end of the desired region
res: Desired resolution. This can be coarser than the resolution of the database when you created it, but not finer.
bin_method: For each bin, you can take the 'mean' (default), 'std', 'max', 'min', or 'range'. I suggest using something besides 'mean' only if absolutely necessary.
transform: When populating the database, the script calculates the min, max, mean, std, log_mean, and log_std of the signal for each track individually, but saves only the raw signal value. It calculates everything else on the fly. Here are the options:
	'none' or None: use raw signal
	'log': log2(signal+1)
	'z': z-transform to have mean 0 and std 1
	'log_z': takes log2(signal+1), then z-transforms it.
Depending on the application, I recommend using either 'log' or 'log_z'. The former will likely be more useful in applications like neural nets, since often the minimum value is assumed to be 0, but if you want to directly compare signal tracks to each other, use 'log_z'. What's very cool is that ChIP-seq tracks (in the mappable regions) are almost always log-normally distributed. This can be explained by biological effects generally being multiplicative, and not additive. It's the same with, e.g., lengths of introns and exons.

For any bug reports, email me at arturj ucla edu
