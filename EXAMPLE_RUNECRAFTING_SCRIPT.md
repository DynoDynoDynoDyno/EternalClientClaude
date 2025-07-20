# Example Script: Runecrafting with Tree-Branch-Leaf Framework

## How It Works Together

### 1. Initialization
When the script starts, the Main class initializes the tree with the RunecraftingBranch, which includes both the BankingLeaf and RunecraftingLeaf.

### 2. Execution Flow
During each loop, the tree checks the validity of the RunecraftingBranch. If the player has Pure essence, it proceeds to evaluate the BankingLeaf and RunecraftingLeaf.
- **BankingLeaf**: Executes if the inventory loadout is not fulfilled (e.g., lacking Pure essence), performing actions to ensure the player is prepared for runecrafting.
- **RunecraftingLeaf**: Executes if the player is ready for runecrafting, handling actions related to interacting with the Runecrafting Altar.

### 3. Looping
After executing the leaves, the script loops back, continually checking conditions and executing appropriate actions until the player runs out of Pure essence.

## Code Implementation

### Main (AbstractScript)
```java
@ScriptManifest(
    name = "Leagues Runecrafter",
    author = "EternalClient",
    description = "Powerlevels runecrafting on leagues",
    category = ScriptCategory.RUNECRAFTING,
    version = 1.0
)
public class Main extends AbstractScript
{

    private final Tree tree = new Tree();

    @Override
    public void onStart(String... args)
    {
        tree.addBranches(
            new RunecraftingBranch().addBranches(
                new BankingLeaf(),
                new RunecraftingLeaf()
            )
        );
    }

    @Override
    public int onLoop()
    {
        // Run our logic tree
        return tree.onLoop();
    }

}
```

### Constants
```java
public class Constants
{
    // Create an ItemVariant since both trimmed and untrimmed has the same functionality
    public static final ItemVariant CRAFTING_CAPE = new ItemVariant(ItemID.CRAFTING_CAPE, ItemID.CRAFTING_CAPET);

    public static final InventoryLoadout runecraftInventoryLoadout = new InventoryLoadout()
        .addReq(ItemID.CRYSTAL_OF_MEMORIES)
        .addReq(ItemID.PURE_ESSENCE, 1, 27);

    public static final EquipmentLoadout runecraftEquipmentLoadout = new EquipmentLoadout()
        .addCape(LeagueUtils.CRAFTING_CAPE);

    public static final int CRAFTING_GUILD_REGION = 11571;

    public static boolean atCraftingGuildBank()
    {
        return LocalPlayer.getRegionID() == CRAFTING_GUILD_REGION;
    }

}
```

### RunecraftingBranch
```java
public class RunecraftingBranch extends Branch
{
    @Override
    public boolean isValid()
    {
        return OwnedItems.contains(ItemID.PURE_ESSENCE);
    }
}
```

### BankingLeaf
```java
public class BankingLeaf extends Leaf
{
    @Override
    public boolean isValid()
    {
        return !Constants.runecraftInventoryLoadout.isFulfilled();
    }

    @Override
    public int onLoop()
    {
        if (Constants.atCraftingGuildBank())
        {
            new WithdrawLoadoutEvent(Constants.runecraftInventoryLoadout, Constants.runecraftEquipmentLoadout).execute();
            return ReactionGenerator.getLowPredictable();
        }

        if (Equipment.contains(CRAFTING_CAPE) && Equipment.forceInteract(EquipmentSlot.CAPE, "Teleport", 2))
        {
            MethodProvider.tickSleepUntil(Constants::atCraftingGuildBank, 10);
            return ReactionGenerator.getNormal();
        }

        if (new InventoryEvent(Inventory.get(Constants.CRAFTING_CAPE), "Teleport").executed())
        {
            MethodProvider.tickSleepUntil(Constants::atCraftingGuildBank, 10);
        }

        return ReactionGenerator.getNormal();
    }
}
```

### RunecraftingLeaf
```java
public class RunecraftingLeaf extends Leaf
{
    @Override
    public boolean isValid()
    {
        return true;
    }

    @Override
    public int onLoop()
    {
        GameObject altar = GameObjects.closest(g -> g.hasName("Altar") && g.hasAction("Craft-rune"));
        if (altar == null)
        {
            if (new InventoryEvent(Inventory.get(ItemID.CRYSTAL_OF_MEMORIES), "Teleport-back").executed())
            {
                MethodProvider.tickSleepUntil(() -> !Constants.atCraftingGuildBank(), 10);
            }

            MethodProvider.tickSleep(1);
            return ReactionGenerator.getLowPredictable();
        }

        if (new EntityInteractEvent(altar, "Craft-rune").executed())
        {
            MethodProvider.tickSleepUntil(() -> !Inventory.contains(ItemID.PURE_ESSENCE), 2);
        }

        return ReactionGenerator.getLowPredictable();
    }
}
```