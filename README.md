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
