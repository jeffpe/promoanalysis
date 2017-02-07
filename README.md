# Count how many subscriptions from the promos are still active and not cancelled
# The next time I do this use SQL Lite or someting like that. This is getting ridiculous.



# load in the required libraries
require "csv"
require "date"

# set the names of the csv files
@promo_file = "all_promo_opportunities.csv"
@current_products_file = "products_this_week_with_ids.csv"
@output_file_name = "promo_analysis.csv"
# set parameters of promo file. First line/field = 0 
@promo_start_line = 1
@promo_promotion = 0
@promo_opp = 2
@promo_product = 3

# set parameters of the current products file
@prods_start_line = 1
@prods_opp = 0
@prods_prod = 5
@prods_access = 6
@prods_status = 7

# final output variables
@current_opp_list = {}
@original_opp_list = {}
@final_promo_hash = {}
@orig_promo_products = Hash.new {|h,k| h[k] = Hash.new(0)}
@orig_promo_count = Hash.new(0)
@current_promo_count = Hash.new(0)
@current_promo_products = Hash.new {|h,k| h[k] = Hash.new(0)}



# Convert CSV to ARRAY, input csv_name, return the array
class Opportunity
	def initialize(opp, prod, access, status, promo)
		@opp = opp
		@prod = prod
		@access = access
		@status = status
		@promo = promo
	end
	
	attr_accessor :opp, :prod, :access, :status, :promo

end



def csv_to_array(csv_name)
	CSV.read(csv_name)
end

# Convert CSV to HASH based on first field, input csv_name, return the hash
def prod_to_hash(csv_name)
	prod_array = CSV.read(csv_name)
	prod_array.each_with_index do |line, index|
		next if line[0] == "Opportunity ID" 
		break if line[0] == nil
		@current_opp_list[line[0]] = Opportunity.new(line[@prods_opp], line[@prods_prod], line[@prods_access], line[@prods_status], "unknown")
		
	end
	
end

def promo_to_hash(csv_name)
	promo_array = CSV.read(csv_name)
	promo_array.each_with_index do |line, index|
		next if line[0] == "Promotion" 
		break if line[0] == nil
		@original_opp_list[line[1]] = Opportunity.new(line[1], line[2], "unknown", "unknown", line[0])
		@orig_promo_count[line[0]] += 1
		@orig_promo_products[line[0]][line[2]] +=1
		

	end

end

def promo_lookup
	@original_opp_list.each_key do |key|
		if @current_opp_list.key?(key) == false
			@original_opp_list[key].access = "Inactive"
			@original_opp_list[key].status = "Cancelled"
		else
			@original_opp_list[key].access = @current_opp_list[key].access
			@original_opp_list[key].status = @current_opp_list[key].status
		end
		# Could attempt to replace this with a select from the hash
		if @original_opp_list[key].access == "Active" && @original_opp_list[key].access != "Cancelled"
			@current_promo_count[@original_opp_list[key].promo] +=1
			@current_promo_products[@original_opp_list[key].promo][@original_opp_list[key].prod] +=1
		end	

	end

end



def csv_output(output_file_name)
	CSV.open(output_file_name, "wb") do |csv|
		csv << ["Promo Output"]
		csv << [Date.today]
		csv << []
		csv << ["Original Promo Counts by Promo"]
		csv << ["PROMO", "ORIGINAL COUNT", "CURRENT COUNT"]
		@orig_promo_count.each do |key, value|
			csv << [key, value, @current_promo_count[key]]
		end
		csv << []
		csv << ["Original Promo Counts by Product"]
		csv << ["PROMO", "PRODUCT", "ORIG COUNT", "CURRENT COUNT"]
		@orig_promo_products.each do |h,k|
			k.each do |v, c|
				csv << [h, v, c, @current_promo_products[h][v]]
			end
		
		end
#################
		csv << []
		csv << ["Current Promo Counts by Promo"]
		csv << ["PROMO", "COUNT"]
		@current_promo_count.each do |key, value|
			csv << [key, value]
		end
		csv << []
		csv << ["Current Promo Counts by Product"]
		csv << ["PROMO", "PRODUCT", "COUNT"]
		@current_promo_products.each do |h,k|
			k.each do |v, c|
				csv << [h, v, c]
			end
		
		end





######################
	end
end


begin
	prod_to_hash(@current_products_file)
	promo_to_hash(@promo_file)
	promo_lookup
	csv_output(@output_file_name)
end


