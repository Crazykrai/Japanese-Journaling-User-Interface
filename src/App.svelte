<script>
    import { Badge } from "$lib/components/ui/badge";
    import { Button } from "$lib/components/ui/button/index.js";
    import * as Card from "$lib/components/ui/card/index.js";
    import * as Dialog from "$lib/components/ui/dialog";
    import { Input } from "$lib/components/ui/input";
    import { Label } from "$lib/components/ui/label/index.js";
    import { Progress } from "$lib/components/ui/progress";
    import * as Table from "$lib/components/ui/table";
    import { Textarea } from "$lib/components/ui/textarea";
    import { toast } from "svelte-sonner";
    import { Toaster } from "$lib/components/ui/sonner";
    import * as RadioGroup from "$lib/components/ui/radio-group";
    import * as Sheet from "$lib/components/ui/sheet/index.js";

    let date = new Date()
    let currentIndex = 0

    let addWordText = ""
    let addHiragana = ""
    let addRomanji = ""
    let addDefinition = ""

    let daysTracked = 2
    let totalWords = 2
    let averageAcc = 80

    let timeGoal = 60
    let accGoal = 90
    let wordGoal = 5

    let entries = [
      {
        tracked: true,
        minutes: 25,
        accuracy: 90,
        difficulty: "easy",
        words: [
          {
            word: "残夢",
            hiragana: "今まで＝いまで",
            romanji: "Zanmu",
            definition: "Dreams that linger in the heart even after waking up."
          },
          {
            word: "狂言",
            hiragana: "今まで＝いまで",
            romanji: "Kyogen",
            definition: "'Mad Words,' a traditional form of Japanese theater"
          },
        ]
      },
      {
        tracked: false,
        minutes: 0,
        accuracy: 0,
        difficulty: "easy",
        words: []
      },
      {
        tracked: true,
        minutes: 35,
        accuracy: 70,
        difficulty: "easy",
        words: []
      },
    ]
    function onSave() {
      if(entries[currentIndex].accuracy != 0 && entries[currentIndex].minutes != 0) {
        entries[currentIndex].tracked = true
        calculateStats()
        toast.success("Successfully saved journal entry for " + date.toDateString())
      } else {
        toast.error("Fill in fields for journal data to save.")
      }
    }

    function addWord() {
      if(addWordText != "" && addHiragana != "" && addRomanji != "" && addDefinition != "") {
        entries[currentIndex].words = [
          ...entries[currentIndex].words,
          {
            word: addWordText,
            hiragana: addHiragana,
            romanji: addRomanji,
            definition: addDefinition
          }
        ]
        addWordText = ""
        addHiragana = ""
        addRomanji = ""
        addDefinition = ""
        calculateStats()
        toast.success("Word successfully added!")
      } 
    }

    function prevDay() {
      if(currentIndex + 1 == entries.length) {
        let newEntry = {
          tracked: true,
          minutes: 0,
          accuracy: 0,
          difficulty: "easy",
          words: []
        }
        entries.push(newEntry)
      }
      currentIndex += 1
      date.setDate(date.getDate() - 1)
      date = date // Needed for Svelte to update DOM 
    }

    function nextDay() {
      console.log(currentIndex)
      if(currentIndex > 0) {
        currentIndex -= 1
        date.setDate(date.getDate() + 1)
        date = date // Needed for Svelte to update DOM 
      }
    }

    function calculateStats() {
      let wordCount = 0
      let dayCount = 0
      let acc = 0
      entries.filter(entry => entry.tracked).map(entry => {
        wordCount += entry.words.length
        dayCount += 1
        // @ts-ignore
        acc += parseInt(entry.accuracy)
      })

      daysTracked = dayCount
      totalWords = wordCount
      averageAcc = Math.round(acc / dayCount * 10) / 10
    }

</script>

<Toaster />
<div class="flex justify-between">
    <Button size="xl" class="text-2xl" on:click={prevDay}> &lt </Button>
    <h1 class="text-4xl pt-2 font-bold">{ date.toDateString() }</h1>
    <Button size="xl" class="justify-end text-2xl" on:click={nextDay}> &gt </Button>
</div>

<div class="flex p-5">
    <h1 class="text-5xl p-10"> Ayush Verma Journaling </h1>
    <Card.Root class="border-2">
        <Card.Header>
          <Card.Title class="text-2xl">Days logged</Card.Title>
        </Card.Header>
        <Card.Content>
          <p class="text-center font-bold text-3xl">{daysTracked}</p>
        </Card.Content>
      </Card.Root>
      <Card.Root class="ml-5 border-2">
        <Card.Header>
          <Card.Title class="text-2xl">Total words logged</Card.Title>
        </Card.Header>
        <Card.Content>
          <p class="text-center font-bold text-3xl">{totalWords}</p>
        </Card.Content>
      </Card.Root>
      <Card.Root class="ml-5 border-2">
        <Card.Header>
          <Card.Title class="text-2xl">Average Accuracy</Card.Title>
        </Card.Header>
        <Card.Content>
          <p class="text-center font-bold text-3xl">{averageAcc}%</p>
        </Card.Content>
      </Card.Root>
</div>

<div class="grid grid-cols-5 gap-10 p-16">
  <div class="pt-6">
    <div class="pb-3">
      <Label for="timeSpent">Minutes Studying <Badge variant={entries[currentIndex].minutes >= timeGoal ? "default" : "destructive"}>{entries[currentIndex].minutes >= timeGoal ? "Goal Met" : "Goal not met"}</Badge> </Label>
      <Input type="number" id="timeSpent" placeholder="0 minutes" bind:value={entries[currentIndex].minutes} />
    </div>
    <div class="pb-3">
      <Label for="accuracy">Accuracy</Label>
      <Input type="number" id="accuracy" placeholder="0%" bind:value={entries[currentIndex].accuracy} />
    </div>
    <div class="pb-3">
      <Label for="difficulty">Today's Difficulty</Label>
      <RadioGroup.Root id="difficulty" bind:value={entries[currentIndex].difficulty}>
        <div class="flex items-center space-x-2">
          <RadioGroup.Item value="easy" id="easy" />
          <Label for="easy">Easy</Label>
        </div>
        <div class="flex items-center space-x-2">
          <RadioGroup.Item value="medium" id="medium" />
          <Label for="medium">Medium</Label>
        </div>
        <div class="flex items-center space-x-2">
          <RadioGroup.Item value="hard" id="hard" />
          <Label for="hard">Hard</Label>
        </div>
      </RadioGroup.Root>
    </div>
    <div class="flex gap-2">
      <Sheet.Root>
        <Sheet.Trigger asChild let:builder>
          <Button builders={[builder]} variant="outline">Set Goals</Button>
        </Sheet.Trigger>
        <Sheet.Content side="left">
          <Sheet.Header>
            <Sheet.Title>Manage your Goals</Sheet.Title>
            <Sheet.Description>
              Make changes to your goals here. Click save when you're done.
            </Sheet.Description>
          </Sheet.Header>
          <div class="grid gap-4 py-4">
            <div class="grid grid-cols-4 items-center gap-4">
              <Label for="timeGoal" class="text-right">Time Spent</Label>
              <Input id="timeGoal" bind:value={timeGoal} class="col-span-3" />
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
              <Label for="accGoal" class="text-right">Average Accuracy</Label>
              <Input id="accGoal" bind:value={accGoal} class="col-span-3" />
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
              <Label for="wordGoal" class="text-right">Total Words</Label>
              <Input id="wordGoal" bind:value={wordGoal} class="col-span-3" />
            </div>
          </div>
          <Sheet.Footer>
            <Sheet.Close asChild let:builder>
              <Button builders={[builder]} type="submit">Save Goals</Button>
            </Sheet.Close>
          </Sheet.Footer>
        </Sheet.Content>
      </Sheet.Root>
      <Button size="default" variant="default" class="" on:click={onSave}>Save</Button>
    </div>
  </div>

  <div class="col-span-3">
    <Table.Root>
      <Table.Caption>Today's words learned so far.</Table.Caption>
      <Table.Header>
        <Table.Row>
          <Table.Head class="w-[100px]">Kanji</Table.Head>
          <Table.Head>Hiragana</Table.Head>
          <Table.Head>Romanji</Table.Head>
          <Table.Head>Definition</Table.Head>
          <Table.Head>
            <Dialog.Root>
              <Dialog.Trigger class="bg-primary text-primary-foreground hover:bg-primary/90 h-11 rounded-full px-4 text-xl">+</Dialog.Trigger>
                <Dialog.Content>
                  <Dialog.Header>
                    <Dialog.Title>Add New Word</Dialog.Title>
                  </Dialog.Header>
                  <div class="pb-3">
                    <Label for="word">Word</Label>
                    <Input type="text" id="word" placeholder="" bind:value={addWordText}/>
                  </div>
                  <div class="pb-3">
                    <Label for="hiragana">Hiragana</Label>
                    <Input type="text" id="hiragana" placeholder="" bind:value={addHiragana} />
                  </div>
                  <div class="pb-3">
                    <Label for="romanji">Romanji</Label>
                    <Input type="text" id="romanji" placeholder="" bind:value={addRomanji} />
                  </div>
                  <div class="pb-3">
                    <Label for="definition">Definition</Label>
                    <Textarea id="definition" placeholder="Meaning of the word" bind:value={addDefinition} />
                  </div>
                  <Dialog.Footer>
                    <Button type="submit" on:click={addWord}>Save</Button>
                  </Dialog.Footer>
                </Dialog.Content>
            </Dialog.Root>
          </Table.Head>
        </Table.Row>
      </Table.Header>
      <Table.Body>
        {#each entries[currentIndex].words as { word, hiragana, romanji, definition}}
          <Table.Row>
            <Table.Cell class="font-medium">{word}</Table.Cell>
            <Table.Cell>{hiragana}</Table.Cell>
            <Table.Cell>{romanji}</Table.Cell>
            <Table.Cell>{definition}</Table.Cell>
          </Table.Row>
        {/each}
      </Table.Body>
    </Table.Root>
  </div>

  <div class="grid grid-cols-2 gap-1">
    <div class="grid justify-end text-right">
      <Label for="prog" class="">Master</Label>
      <Label for="prog" class="">Expert</Label>
      <Label for="prog" class="">Proficient</Label>
      <Label for="prog" class="">Basic</Label>
    </div>
    <div>
      <Progress class="h-full w-1/6" style="transform:rotate(180deg)" value=30 id="prog" />
    </div>
  </div>
  
</div>
