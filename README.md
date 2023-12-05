# Advent of Code 2023

(I am not going for the global leaderboard this year. My workflow is roughly the same as previous year.)

## Day 1

```ruby
# Day 1, Part 1
p [*$<].sum { x = _1.scan(/\d/); (x.first + x.last).to_i }

# Day 1, Part 2
ns = %w(one two three four five six seven eight nine)
p [*$<].sum {
  x = _1.gsub(/one|two|three|four|five|six|seven|eight|nine/){ns.index($&) + 1 }.scan(/\d/)
  y = _1.reverse.gsub(/eno|owt|eerht|ruof|evif|xis|neves|thgie|enin/){ns.index($&.reverse) + 1 }.scan(/\d/).reverse
  (x.first + y.last).to_i
}
```

## Day 2

```ruby
# Day 2, Parsing
games = [*$<].map { |l|
  l =~ /Game (\d+)/
  game_id = $1.to_i
  sets = l.split(': ')[1].split('; ').map {
    _1.split(', ').map(&:split).map(&:reverse).to_h
  }
  [game_id, sets]
}.to_h

# Day 2, Part 1
pp games.filter { |id, sets|
  sets.all? { |set|
    c = -> (k) { (set[k] || 0).to_i }
    c['red'] <= 12 && c['green'] <= 13 && c['blue'] <= 14
  }
}.keys.sum

# Day 2, Part 2
pp games.sum { |id, sets|
  ['red', 'green', 'blue'].map { |k|
    sets.map { |set| (set[k] || 0).to_i }.max }.inject(&:*)
}
```

## Day 3

```ruby
# Day 3, Parsing
matrix = [*$<].map(&:strip).map(&:chars)
hash = Hash.new('.')
matrix.each_with_index do |row, y|
  row.each_with_index do |cell, x|
    hash[[y, x]] = cell
  end
end

# Day 3, Part 1
sum = 0
matrix.each_with_index do |row, y|
  row.join.scan(/\d+/) {
    num = $&.to_i
    x_start = $`.size
    x_end = x_start + $&.size - 1
    o = []
    (y-1..y+1).each do |yy|
      (x_start-1..x_end+1).each do |xx|
        o << hash[[yy, xx]]
      end
    end
    sum += num if o.join.gsub(/[\d\.]/, '').size > 0
  }
end
p sum

# Day 3, Part 2
numbers_next_to_stars = []
matrix.each_with_index do |row, y|
  row.join.scan(/\d+/) {
    num = $&.to_i
    x_start = $`.size
    x_end = x_start + $&.size - 1
    (y-1..y+1).each do |yy|
      (x_start-1..x_end+1).each do |xx|
        if hash[[yy, xx]] == '*'
          numbers_next_to_stars << [num, [yy, xx]]
        end
      end
    end
  }
end
pp numbers_next_to_stars.group_by(&:last)
  .filter { |k, v| v.size == 2 }
  .sum { |k, v| v.map(&:first).inject(&:*) }
```

## Day 4

```ruby
# Day 4
class Card < Struct.new(:id, :winning_numbers, :numbers_you_have)
  def points
    m = matching_numbers
    m == 0 ? 0 : 2 ** (m - 1)
  end
  def matching_numbers
    (winning_numbers & numbers_you_have).size
  end
end

# Day 4, Parsing
cards = [*$<].map { |line|
  left, right = line.split('|')
  id, *winning_numbers = left.scan(/\d+/).map(&:to_i)
  numbers_you_have = right.scan(/\d+/).map(&:to_i)
  Card.new(id, winning_numbers, numbers_you_have)
}

# Day 4, Part 1
p cards.sum(&:points)

# Day 4, Part 2
copies = cards.map { 1 }
cards.each_with_index { |card, i|
  wins = card.matching_numbers
  (1..wins).each { |j|
    copies[i + j] += copies[i]
  }
}
p copies.sum
```

## Day 5

```ruby
# Day 5
p seed_numbers = gets.scan(/\d+/).map(&:to_i)

class ConversionRule < Struct.new(:range, :target_start)
  def target_range
    @target_range ||= target_start...(target_start + range.size)
  end
end

class Map < Struct.new(:rules)
  def convert(n)
    found_rule = rules.find { |r| r.range.include?(n) }
    found_rule ? found_rule.target_start + n - found_rule.range.begin : n
  end
  def inverse_convert(n)
    found_rule = rules.find { |r| r.target_range.include?(n) }
    found_rule ? found_rule.range.begin + n - found_rule.target_start : n
  end
end

maps = $<.read.split(/\n\s*\n/).map { |f|
  Map.new(f.scan(/\d+/).map(&:to_i).each_slice(3).map { |a, b, c|
    ConversionRule.new(b...b + c, a)
  })
}

# Day 5, Part 1
p seed_numbers.map { |n| maps.inject(n) { |n, m| m.convert(n) } }.min

# Day 5, Part 2, brute force (doesn’t finish)
ranges = seed_numbers.each_slice(2).map { |a, b| a...a + b }
min_l = nil
last_time = 0.0
processed = 0
total = ranges.sum(&:size)
ranges.each do |r|
  r.each do |n|
    location = maps.inject(n) { |n, m| m.convert(n) }
    processed += 1
    min_l = location if min_l.nil? || location < min_l
    if Time.now.to_f - last_time > 1.0
      puts "Current min: #{min_l}, processed: #{processed}, total: #{total}"
      last_time = Time.now.to_f
    end
  end
end
p min_l

# Day 5, Part 2
ranges = seed_numbers.each_slice(2).map { |a, b| a...a + b }
interesting_numbers = []

maps.reverse_each do |map|
  interesting_numbers += map.rules.map(&:target_start)
  interesting_numbers = interesting_numbers.sort.uniq
  interesting_numbers = interesting_numbers.map { |n| map.inverse_convert(n) }
  p interesting_numbers
end

min_l = nil
ranges.each do |r|
  rr = [r.begin] + interesting_numbers.select { |n| r.include?(n) }
  rr.each do |n|
    location = maps.inject(n) { |n, m| m.convert(n) }
    puts "#{n} => #{location}"
    if min_l.nil? || location < min_l
      min_l = location
      p min_l
    end
  end
end

p min_l
```
