<h1 align = "center"> Mine Mine no Mi Modded </h1>

<p align = "center"> The Mine Mine no Mi mod already features a lot of API endpoints and possibilites to extend it. </p>
<p align = "center"> Sadly this is not enough for some Applications of mine and some Endpoints are straight up wrong, not implemented or missing. </p>
<p align = "center"> This version features changes made by me. A diff of changes can be read below. </p>
<p align = "center"> I am sadly not allowed to release the source code as the creator "Wynd" does not wish for it to be open to the public. </p>

<br>
<br>
<h2 align = "center"> Commit Log of changes made by myself </h2>

```log
commit 4c3f2e006b0d5fffd0bd2bfae4cbf889c3bd0956
Author: DerHammerclock <rathmerdominik@outlook.de>
Date:   Thu Nov 23 16:20:12 2023 +0100

    feat(API): Add multiple OneFruit Events

commit f00e1ba7a595e40eca090263f295f2f0994cc86a
Author: DerHammerclock <rathmerdominik@outlook.de>
Date:   Tue Nov 14 09:21:09 2023 +0100

    fix(ForgeSetup): Migrate existing Crews to add the new Creation Date

commit 20a6450937e028dadd0d5b6257f9661bc062acea
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Mon Nov 13 00:53:22 2023 +0100

    feat(Crew): Added Crew creation date

commit ec2d9cfe2a081c1f8d887669dcdeed65b8970212
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Sat Nov 4 18:59:34 2023 +0100

    fix(CCreateCrewPacket): Actually use the Crew Create event

commit 7d5cf0c2dee2883f934032ccc3ab41d562bef31f
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Fri Nov 3 23:32:52 2023 +0100

    feat(CrewEvent): Add new Kick event when kicking a crew member

commit c06fcbafd173348b42b3c3834c8f25aaac441886
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Fri Nov 3 18:22:00 2023 +0100

    fix(JollyRoger): Replace current pixel replacer with tinting

commit 6b1a997983384eebdb991bf5d04a7327d90abfb7
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Thu Nov 2 16:20:26 2023 +0100

    feat(JollyRogerEvent): Added JollyRoger Update event

commit a193cf1f1594eb764317c0d1261abe1932c51b44
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Thu Nov 2 15:36:54 2023 +0100

    fix(JollyRoger): Removed a debug print statement

commit 00af82773dd855465460734bc8ede2e5bd44d5aa
Author: rathmerdominik <rathmerdominik@outlook.de>
Date:   Wed Nov 1 18:08:02 2023 +0100

    feat(JollyRoger): Added possibility to get the JollyRoger as an Image
```

<br>
<br>
<h2 align = "center"> Diff of Changes made to the mod </h2>

`src/main/java/xyz/pixelatedw/mineminenomi/api/crew/Crew.java`
```diff
 package xyz.pixelatedw.mineminenomi.api.crew;
 
+import java.time.Instant;
 import java.util.ArrayList;
 import java.util.List;
 import java.util.Optional;
@@ -17,6 +18,7 @@ public class Crew
 {
 	private String name;
 	private boolean isTemporary;
+	private long creationDate;
 	private List<Member> members = new ArrayList<Member>();
 	private JollyRoger jollyRoger = new JollyRoger();
 	
@@ -24,7 +26,7 @@ public class Crew
 
 	public Crew(String name, LivingEntity entity)
 	{
-		this(name, entity.getUUID(), entity.getDisplayName().getString());
+		this(name, entity.getUUID(), entity.getDisplayName().getString(), Instant.now().getEpochSecond());
 	}
 	
 	public static Crew from(CompoundNBT nbt)
@@ -34,11 +36,12 @@ public class Crew
 		return crew;
 	}
 	
-	public Crew(String name, UUID capId, String username)
+	public Crew(String name, UUID capId, String username, long creationDate)
 	{
 		this.name = name;
 		this.isTemporary = true;
 		this.addMember(capId, username).setIsCaptain(true);
+		this.creationDate = creationDate;
 	}
 
 	public void setName(String name)
@@ -126,6 +129,14 @@ public class Crew
 	{
 		this.jollyRoger = jr;
 	}
+
+	public long getCreationDate() {
+		return this.creationDate;
+	}
+	
+	public void setCreationDate(long creationDate) {
+		this.creationDate = creationDate;
+	}
 	
 	public CompoundNBT write()
 	{
@@ -145,6 +156,8 @@ public class Crew
 
 		CompoundNBT jollyRogerData = this.jollyRoger.write();
 		crewNBT.put("jollyRoger", jollyRogerData);
+
+		crewNBT.putLong("creationDate", this.creationDate);
 		
 		return crewNBT;
 	}
@@ -164,6 +177,13 @@ public class Crew
 		
 		CompoundNBT jollyRogerData = nbt.getCompound("jollyRoger");
 		this.jollyRoger.read(jollyRogerData);
+
+		Long creationDate = nbt.getLong("creationDate");
+		if(creationDate == 0L) {
+			this.creationDate = Instant.now().getEpochSecond();
+			return;
+		}
+		this.creationDate = creationDate;
 	}
 
 	public static class Member
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/crew/JollyRoger.java`

```diff
 package xyz.pixelatedw.mineminenomi.api.crew;
 
+import java.awt.Color;
+import java.awt.image.BufferedImage;
+import java.io.IOException;
 import java.util.Arrays;
+import java.util.Optional;
+
+import javax.imageio.ImageIO;
 
 import net.minecraft.nbt.CompoundNBT;
 import net.minecraft.nbt.ListNBT;
@@ -9,6 +15,7 @@ import net.minecraftforge.common.util.Constants;
 import net.minecraftforge.fml.common.registry.GameRegistry;
 import xyz.pixelatedw.mineminenomi.ModMain;
 import xyz.pixelatedw.mineminenomi.init.ModJollyRogers;
+import xyz.pixelatedw.mineminenomi.wypi.WyHelper;
 
 public class JollyRoger
 {
@@ -254,4 +261,117 @@ public class JollyRoger
 	{
 		return Arrays.stream(this.details).parallel().anyMatch(detail -> detail != null && detail.equals(det));
 	}
+
+	/**
+	 * Returns the JollyRoger as a BufferedImage.
+	 *
+	 * Returns Optional.empty() if there have been an IOError.
+	 *
+	 * @return Optional<BufferedImage>
+	 */
+	public Optional<BufferedImage> getAsBufferedImage()
+	{
+		try {
+			BufferedImage jollyRogerImage = new BufferedImage(128, 128, BufferedImage.TYPE_INT_ARGB);
+
+			for (JollyRogerElement backgroundElement : this.backgrounds)
+			{
+				if (backgroundElement == null)
+				{
+					continue;
+				}
+
+				BufferedImage backgroundElementImage = this.elementToImage(backgroundElement);
+
+				jollyRogerImage.getGraphics().drawImage(backgroundElementImage, 0, 0, null);
+			}
+
+			BufferedImage jollyRogerBase = this.elementToImage(this.base);
+			jollyRogerImage.getGraphics().drawImage(jollyRogerBase, 0, 0, null);
+
+			for (JollyRogerElement detailElement : this.details)
+			{
+				if (detailElement == null)
+				{
+					continue;
+				}
+
+				BufferedImage detailElementImage = this.elementToImage(detailElement);
+
+				jollyRogerImage.getGraphics().drawImage(detailElementImage, 0, 0, null);
+			}
+
+			return Optional
+					.of(jollyRogerImage);
+		} catch (IOException e)
+		{
+			ModMain.LOGGER.error(e.getMessage());
+		}
+
+		return Optional.empty();
+	}
+
+	/**
+	 * Generates a BufferedImage with coloring if necessary out of a
+	 * JollyRogerElement
+	 *
+	 * @param element
+	 * @return BufferedImage
+	 * @throws IOException
+	 */
+	private BufferedImage elementToImage(JollyRogerElement element) throws IOException
+	{
+		String assetPath = "assets/mineminenomi/";
+
+		BufferedImage elementImage = ImageIO.read(getClass().getClassLoader()
+				.getResourceAsStream(assetPath + element.getTexture().getPath()));
+
+		if (element.canBeColored())
+		{
+			elementImage = this.applyColorToImage(element.getColor(),
+					elementImage);
+		}
+
+		return elementImage;
+	}
+
+	/**
+	 * Applies a color to a given image with the given hex value.
+	 * Skips pixels with no Alpha value or full black pixel.
+	 *
+	 * @param hex
+	 * @param element
+	 * @return BufferedImage
+	 */
+	private BufferedImage applyColorToImage(String hex, BufferedImage image)
+	{
+		Color color = WyHelper.hexToRGB(hex);
+
+		for (int x = 0; x < image.getWidth(); x++)
+		{
+			for (int y = 0; y < image.getHeight(); y++)
+			{
+				int rgba = image.getRGB(x, y);
+				Color pixelColor = new Color(rgba, true);
+
+				if (pixelColor.getAlpha() != 0 && (rgba & 0x00FFFFFF) != 0)
+				{
+					Integer tintedPixel = tintABGRPixel(pixelColor.getRGB(), color);
+					image.setRGB(x, y, tintedPixel);
+				}
+			}
+		}
+
+		return image;
+	}
+
+	public static Integer tintABGRPixel(int pixelColor, Color tintColor) {
+		int x = pixelColor>>16 & 0xff, y = pixelColor>>8 & 0xff, z = pixelColor & 0xff;
+		int top = 2126*x + 7252*y + 722*z;
+		int Btemp = (int)((tintColor.getBlue() * top * 1766117501L) >> 52);
+		int Gtemp = (int)((tintColor.getGreen() * top * 1766117501L) >> 52);
+		int Rtemp = (int)((tintColor.getRed() * top * 1766117501L) >> 52);
+	
+		return ((pixelColor>>24 & 0xff) << 24) | Btemp & 0xff | (Gtemp & 0xff) << 8 | (Rtemp & 0xff) << 16;
+	}
 }
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/CrewEvent.java`

```diff
 			super(player, crew);
 		}
 	}
+
+	@Cancelable 
+	public static class Kick extends CrewEvent
+	{
+		public Kick(PlayerEntity player, Crew crew)
+		{
+			super(player, crew);
+		}
+	}
 }
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/JollyRogerEvent.java`

```diff
+package xyz.pixelatedw.mineminenomi.api.events;
+
+import net.minecraftforge.eventbus.api.Event;
+import xyz.pixelatedw.mineminenomi.api.crew.Crew;
+import xyz.pixelatedw.mineminenomi.api.crew.JollyRoger;
+
+public class JollyRogerEvent extends Event
+{
+	private JollyRoger jollyRoger;
+	private Crew crew;
+
+	public JollyRogerEvent(JollyRoger jollyRoger, Crew crew)
+	{
+		this.jollyRoger = jollyRoger;
+		this.crew = crew;
+	}
+
+	public Crew getCrew()
+	{
+		return this.crew;
+	}
+
+	public JollyRoger getJollyRoger()
+	{
+		return this.jollyRoger;
+	}
+	public static class Update extends JollyRogerEvent
+	{
+		public Update(JollyRoger jollyRoger, Crew crew)
+		{
+			super(jollyRoger, crew);
+		}
+	}
+}
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/onefruit/DroppedDevilFruitEvent.java`

```diff
+package xyz.pixelatedw.mineminenomi.api.events.onefruit;
+
+import javax.annotation.Nonnull;
+import javax.annotation.Nullable;
+import net.minecraft.entity.LivingEntity;
+import net.minecraft.item.Item;
+import net.minecraftforge.event.entity.EntityEvent;
+
+public class DroppedDevilFruitEvent extends EntityEvent {
+	private final Item devilFruit;
+	private final String reason;
+
+	public DroppedDevilFruitEvent(@Nullable LivingEntity entity, @Nonnull Item devilFruit,
+			String reason) {
+		super(entity);
+		this.devilFruit = devilFruit;
+		this.reason = reason;
+	}
+
+	public Item getItem() {
+		return this.devilFruit;
+	}
+
+	public String getReason() {
+		return this.reason;
+	}
+}
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/EatDevilFruitEvent.java`
```diff
rename from src/main/java/xyz/pixelatedw/mineminenomi/api/events/EatDevilFruitEvent.java
rename to src/main/java/xyz/pixelatedw/mineminenomi/api/events/onefruit/EatDevilFruitEvent.java
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/onefruit/EatDevilFruitEvent.java`
```diff
+package xyz.pixelatedw.mineminenomi.api.events.onefruit;
 
 import javax.annotation.Nonnull;
 
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/onefruit/InventoryDevilFruitEvent.java`

```diff
+package xyz.pixelatedw.mineminenomi.api.events.onefruit;
+
+import javax.annotation.Nonnull;
+import net.minecraft.entity.LivingEntity;
+import net.minecraft.item.Item;
+import net.minecraftforge.event.entity.EntityEvent;
+
+public class InventoryDevilFruitEvent extends EntityEvent {
+private final Item devilFruit;
+	private final String reason;
+
+	public InventoryDevilFruitEvent(LivingEntity entity, @Nonnull Item devilFruit,
+			String reason) {
+		super(entity);
+		this.devilFruit = devilFruit;
+		this.reason = reason;
+	}
+
+	public Item getItem() {
+		return this.devilFruit;
+	}
+
+	public String getReason() {
+		return this.reason;
+	}
+}
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/events/onefruit/LostDevilFruitEvent.java`

```diff
+package xyz.pixelatedw.mineminenomi.api.events.onefruit;
+
+import javax.annotation.Nonnull;
+import net.minecraft.entity.LivingEntity;
+import net.minecraft.item.Item;
+import net.minecraftforge.event.entity.EntityEvent;
+
+public class LostDevilFruitEvent extends EntityEvent{
+	private final String reason;
+	private final Item devilFruit;
+
+	public LostDevilFruitEvent(LivingEntity entity, @Nonnull Item devilFruit,
+			String reason) {
+		super(entity);
+		this.devilFruit = devilFruit;
+		this.reason = reason;
+	}
+
+	public Item getItem() {
+		return this.devilFruit;
+	}
+
+	public String getReason() {
+		return this.reason;
+	}
+}
```

`src/main/java/xyz/pixelatedw/mineminenomi/api/helpers/DevilFruitHelper.java`

```diff
 import net.minecraft.util.math.vector.Vector3d;
 import net.minecraft.world.World;
 import net.minecraft.world.server.ServerWorld;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.registries.ForgeRegistries;
 import xyz.pixelatedw.mineminenomi.ModMain;
 import xyz.pixelatedw.mineminenomi.api.OneFruitEntry;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.DroppedDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.InventoryDevilFruitEvent;
 import xyz.pixelatedw.mineminenomi.config.CommonConfig;
 import xyz.pixelatedw.mineminenomi.config.GeneralConfig;
 import xyz.pixelatedw.mineminenomi.data.entity.devilfruit.DevilFruitCapability;
@@ -103,24 +106,30 @@ public class DevilFruitHelper
 
 		List<ItemEntity> droppedItems = WyHelper.getNearbyEntities(entity.blockPosition(), entity.level, REINCARNATION_RANGE, null, ItemEntity.class);
 		droppedItems.removeIf(entry -> !set.contains(entry.getItem().getItem()));
-		if (!droppedItems.isEmpty() && chance <= CommonConfig.INSTANCE.getChanceForDroppedAppleReincarnation())
+		if (!droppedItems.isEmpty() && chance <= CommonConfig.INSTANCE.getChanceForDroppedAppleReincarnation()) 
 		{
 			AkumaNoMiItem fruit = (AkumaNoMiItem) DevilFruitHelper.getDevilFruitItem(props.getDevilFruit());
+			
 			if (CommonConfig.INSTANCE.getRandomizedFruits()) {
-				fruit = fruit.getReverseShiftedFruit(entity.level);				
+				fruit = fruit.getReverseShiftedFruit(entity.level);
 			}
 			droppedItems.get(0).setItem(new ItemStack(fruit));
 			worldData.updateOneFruit(fruit.getFruitKey(), null, OneFruitEntry.Status.DROPPED, "Reincarnated when " + entity.getDisplayName().getString() + " died", true);
+
+			DroppedDevilFruitEvent droppedEvent = new DroppedDevilFruitEvent((PlayerEntity) entity, fruit, "Reincarnated when " + entity.getDisplayName().getString() + " died");
+			MinecraftForge.EVENT_BUS.post(droppedEvent);
+
 			return true;
 		}
 
 		List<PlayerEntity> players = WyHelper.getNearbyPlayers(entity.blockPosition(), entity.level, REINCARNATION_RANGE, null);
 		players.removeIf(entry -> !entry.inventory.hasAnyOf(set));
+
 		if (!players.isEmpty() && chance <= CommonConfig.INSTANCE.getChanceForInventoryAppleReincarnation())
 		{
 			boolean flag = setFruitInInv(players.get(0).inventory, players.get(0), entity, entity.level, props.getDevilFruit());
 			if (flag) {
-				return true;				
+				return true;
 			}
 		}
 
@@ -160,6 +169,7 @@ public class DevilFruitHelper
 	{
 		if (inv == null)
 			return false;
+
 		int stackIndex = WyHelper.getIndexOfItemStack(Items.APPLE, inv);
 		if (stackIndex != -1)
 		{
@@ -169,6 +179,9 @@ public class DevilFruitHelper
 			inv.setItem(stackIndex, new ItemStack(fruit));
 			UUID invOwnerUUID = invOwner != null ? invOwner.getUUID() : null;
 			ExtendedWorldData.get(level).updateOneFruit(fruit.getFruitKey(), invOwnerUUID, OneFruitEntry.Status.INVENTORY, "Reincarnated in " + entity.getDisplayName().getString() + "'s inventory", true);
+
+			InventoryDevilFruitEvent postEvent = new InventoryDevilFruitEvent(invOwner, DevilFruitHelper.getDevilFruitItem(df), "Reincarnated in " + entity.getDisplayName().getString() + "'s inventory");
+			MinecraftForge.EVENT_BUS.post(postEvent);
 			return true;
 		}
 
```

`src/main/java/xyz/pixelatedw/mineminenomi/commands/RemoveDFCommand.java`

```diff
 			IAbilityData abilityDataProps = AbilityDataCapability.get(player);
 			ExtendedWorldData worldData = ExtendedWorldData.get(player.level);
 	
-			worldData.lostOneFruit(devilFruitProps.getDevilFruit(), player.getUUID(), "Removed via Command");
+			worldData.lostOneFruit(devilFruitProps.getDevilFruit(), player, "Removed via Command");
 			if(devilFruitProps.hasYamiPower()) {
-				worldData.lostOneFruit("yami_yami", player.getUUID(), "Removed via Command");
+				worldData.lostOneFruit("yami_yami", player, "Removed via Command");
 			}
 			
 			devilFruitProps.removeDevilFruit();
```

`src/main/java/xyz/pixelatedw/mineminenomi/data/world/ExtendedWorldData.java`

```diff
@@ -12,6 +12,7 @@ import javax.annotation.Nullable;
 
 import com.google.common.base.Strings;
 
+import net.minecraft.entity.Entity;
 import net.minecraft.entity.LivingEntity;
 import net.minecraft.nbt.CompoundNBT;
 import net.minecraft.nbt.ListNBT;
@@ -20,6 +21,7 @@ import net.minecraft.world.IWorld;
 import net.minecraft.world.World;
 import net.minecraft.world.server.ServerWorld;
 import net.minecraft.world.storage.WorldSavedData;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.common.util.Constants;
 import net.minecraftforge.fml.server.ServerLifecycleHooks;
 import xyz.pixelatedw.mineminenomi.ModMain;
@@ -28,6 +30,9 @@ import xyz.pixelatedw.mineminenomi.api.SoulboundMark;
 import xyz.pixelatedw.mineminenomi.api.crew.Crew;
 import xyz.pixelatedw.mineminenomi.api.crew.Crew.Member;
 import xyz.pixelatedw.mineminenomi.api.crew.JollyRoger;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.InventoryDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.LostDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.helpers.DevilFruitHelper;
 import xyz.pixelatedw.mineminenomi.api.protection.ProtectedArea;
 import xyz.pixelatedw.mineminenomi.wypi.WyHelper;
 
@@ -341,7 +346,7 @@ public class ExtendedWorldData extends WorldSavedData
 				// If the fruit already exists in somebody's inventory and another user tries to eat it, count it as a dupe
 				if(oneFruit.get().getStatus() == OneFruitEntry.Status.INVENTORY && status == OneFruitEntry.Status.IN_USE && !sameOwner)
 					return false;				
-			}
+		}
 			
 			if(!Strings.isNullOrEmpty(message))
 				oneFruit.get().setStatusMessage(message);
@@ -352,14 +357,17 @@ public class ExtendedWorldData extends WorldSavedData
 		return true;
 	}
 	
-	public void lostOneFruit(String key, UUID uuid, String message)
+	public void lostOneFruit(String key, @Nullable LivingEntity entity, String message)
 	{
 		Optional<OneFruitEntry> oneFruit = this.oneFruit.stream().filter((entry) -> entry.getKey().equals(key)).findFirst();
 		if(oneFruit.isPresent())
 		{
 			oneFruit.get().setStatusMessage(message);
-			oneFruit.get().update(uuid, OneFruitEntry.Status.LOST);
+			oneFruit.get().update(entity.getUUID(), OneFruitEntry.Status.LOST);
 			this.setDirty();
+			
+			LostDevilFruitEvent lostEvent = new LostDevilFruitEvent(entity, DevilFruitHelper.getDevilFruitItem(oneFruit.get().getKey()), message);
+			MinecraftForge.EVENT_BUS.post(lostEvent);
 		}
 	}
 	
```

`src/main/java/xyz/pixelatedw/mineminenomi/events/devilfruits/OneFruitPerWorldEvents.java`

```diff
 import java.util.ArrayList;
 import java.util.Date;
 import java.util.List;
-
+import java.util.UUID;
 import net.minecraft.block.LeavesBlock;
 import net.minecraft.client.Minecraft;
 import net.minecraft.client.gui.screen.inventory.ContainerScreen;
@@ -32,6 +32,7 @@ import net.minecraft.world.World;
 import net.minecraftforge.api.distmarker.Dist;
 import net.minecraftforge.api.distmarker.OnlyIn;
 import net.minecraftforge.client.event.GuiScreenEvent;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.common.util.Constants;
 import net.minecraftforge.event.entity.EntityJoinWorldEvent;
 import net.minecraftforge.event.entity.item.ItemExpireEvent;
@@ -47,6 +48,9 @@ import net.minecraftforge.registries.ForgeRegistries;
 import xyz.pixelatedw.mineminenomi.ModMain;
 import xyz.pixelatedw.mineminenomi.api.OneFruitEntry;
 import xyz.pixelatedw.mineminenomi.api.abilities.AbilityCategory;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.DroppedDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.InventoryDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.LostDevilFruitEvent;
 import xyz.pixelatedw.mineminenomi.api.helpers.DevilFruitHelper;
 import xyz.pixelatedw.mineminenomi.config.CommonConfig;
 import xyz.pixelatedw.mineminenomi.data.entity.ability.AbilityDataCapability;
@@ -92,9 +96,12 @@ public class OneFruitPerWorldEvents
 
 				if (df != null)
 				{
-					boolean flag = ExtendedWorldData.get(event.getWorld()).updateOneFruit(df.getFruitKey(), null, OneFruitEntry.Status.DROPPED);
+					boolean flag = ExtendedWorldData.get(event.getWorld()).updateOneFruit(df.getFruitKey(), null, OneFruitEntry.Status.DROPPED, "Dropped from leaves. Sheared by " + event.getPlayer().getDisplayName());
 					if(flag)
 						event.getWorld().addFreshEntity(new ItemEntity((World) event.getWorld(), event.getPos().getX(), event.getPos().getY(), event.getPos().getZ(), new ItemStack(df)));
+					
+					DroppedDevilFruitEvent postEvent = new DroppedDevilFruitEvent(event.getPlayer(), df, "Dropped from leaves. Sheared by " + event.getPlayer().getDisplayName());
+					MinecraftForge.EVENT_BUS.post(postEvent);
 				}
 			}
 		}
@@ -212,7 +219,7 @@ public class OneFruitPerWorldEvents
 			PlayerEntity player = (PlayerEntity) event.getEntityLiving();
 			IDevilFruit fruitProps = DevilFruitCapability.get(player);
 			ExtendedWorldData worldData = ExtendedWorldData.get(player.level);
-			
+
 			IDevilFruit props = DevilFruitCapability.get(player);
 			boolean fruitRespawned = DevilFruitHelper.respawnDevilFruit(player, props);
 
@@ -223,7 +230,7 @@ public class OneFruitPerWorldEvents
 				// circulation
 				if (!fruitProps.hasDevilFruit(ModAbilities.YOMI_YOMI_NO_MI) || YomiMorphInfo.INSTANCE.isActive(player)) {
 					if (DevilFruitHelper.canDevilFruitRespawn()) {
-						worldData.lostOneFruit(fruitProps.getDevilFruit(), player.getUUID(), "User's death");
+						worldData.lostOneFruit(fruitProps.getDevilFruit(), player, "User died");
 					}
 				}
 
@@ -231,10 +238,10 @@ public class OneFruitPerWorldEvents
 				// from circulation
 				if (fruitProps.hasYamiPower()) {
 					if (DevilFruitHelper.canDevilFruitRespawn()) {
-						worldData.lostOneFruit(ModAbilities.YAMI_YAMI_NO_MI.getFruitKey(), player.getUUID(), "User's death");
+						worldData.lostOneFruit(ModAbilities.YAMI_YAMI_NO_MI.getFruitKey(), player, "User died");
 					}
 				}
-				
+
 				ArrayList<ItemStack> slots = new ArrayList<ItemStack>();
 				slots.addAll(player.inventory.items);
 				slots.addAll(player.inventory.offhand);
@@ -244,10 +251,13 @@ public class OneFruitPerWorldEvents
 					{
 						String key = ((AkumaNoMiItem) invStack.getItem()).getFruitKey();
 						if(worldData.isFruitInUse(key)) {
-							invStack.shrink(invStack.getCount());							
+							invStack.shrink(invStack.getCount());
 						}
 						else {
-							worldData.updateOneFruit(key, player.getUUID(), OneFruitEntry.Status.DROPPED);							
+							worldData.updateOneFruit(key, player.getUUID(), OneFruitEntry.Status.DROPPED);
+
+							DroppedDevilFruitEvent postEvent = new DroppedDevilFruitEvent(player, DevilFruitHelper.getDevilFruitItem(props.getDevilFruit()), "User died");
+							MinecraftForge.EVENT_BUS.post(postEvent);
 						}
 					}
 				}
@@ -281,6 +291,7 @@ public class OneFruitPerWorldEvents
 						// If the player who joins is not the one owning the DF (hopefully due to away time) remove their fruit
 						boolean somebodyElseHasFruit = entry.getOwner().isPresent() && !entry.getOwner().get().equals(player.getUUID());
 						boolean nobodyHasFruit = !entry.getOwner().isPresent() && entry.getStatus() == OneFruitEntry.Status.LOST;
+
 						if(somebodyElseHasFruit || nobodyHasFruit)
 						{
 							if (entry.getKey().equals(fruitData.getDevilFruit()))
@@ -383,8 +394,10 @@ public class OneFruitPerWorldEvents
 					else
 					{
 						ExtendedWorldData worldProps = ExtendedWorldData.get(player.level);
-						boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), player.getUUID(), OneFruitEntry.Status.INVENTORY);
+						boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), player.getUUID(), OneFruitEntry.Status.INVENTORY, "Picked up from ground");
 						event.setCanceled(!flag);
+						InventoryDevilFruitEvent inventoryEvent = new InventoryDevilFruitEvent(player, stack.getItem(), "Picked up from ground");
+						MinecraftForge.EVENT_BUS.post(inventoryEvent);
 					}
 				}
 				else
@@ -395,14 +408,18 @@ public class OneFruitPerWorldEvents
 			else {
 				if(stack.getItem() instanceof AkumaNoMiItem) {
 					ExtendedWorldData worldProps = ExtendedWorldData.get(player.level);
-					boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), player.getUUID(), OneFruitEntry.Status.INVENTORY);
+					boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), player.getUUID(), OneFruitEntry.Status.INVENTORY, "Picked up from ground");
 					event.setCanceled(!flag);
+
+					InventoryDevilFruitEvent inventoryEvent = new InventoryDevilFruitEvent(player, stack.getItem(), "Picked up from ground");
+					MinecraftForge.EVENT_BUS.post(inventoryEvent);
+
 					OneFruitPerWorldEvents.checkPlayerInventory(player);
 				}
 			}
 		}
 	}
-	
+
 	@SubscribeEvent
 	public static void onDevilFruitDropped(EntityJoinWorldEvent event) {
 		if (CommonConfig.INSTANCE.hasOneFruitPerWorldSimpleLogic())
@@ -410,15 +427,28 @@ public class OneFruitPerWorldEvents
 			if(event.getEntity() instanceof ItemEntity) {
 				ItemEntity entity = ((ItemEntity)event.getEntity());
 				ItemStack stack = entity.getItem();
+				UUID thrower = entity.getThrower();
+
+				PlayerEntity player = null;
+				if(thrower != null){
+					player = event.getWorld().getPlayerByUUID(thrower);
+				}
+
 				if(!stack.isEmpty() && stack.getItem() instanceof AkumaNoMiItem) {
 					ExtendedWorldData worldProps = ExtendedWorldData.get(event.getWorld());
-					boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), entity.getThrower(), OneFruitEntry.Status.DROPPED, null, true);
-					event.setCanceled(!flag);					
+				
+					boolean flag = worldProps.updateOneFruit(((AkumaNoMiItem) stack.getItem()).getFruitKey(), entity.getThrower(), OneFruitEntry.Status.DROPPED, "Fruit got added to world " + (player == null ? "" : "by " + player.getDisplayName().getString()), true);
+
+					event.setCanceled(!flag);
+
+					// Player might be null but that is fine
+					DroppedDevilFruitEvent postEvent = new DroppedDevilFruitEvent(player, stack.getItem(), "Fruit got added to world " + (player == null ? "" : "by " + player.getDisplayName().getString()));
+					MinecraftForge.EVENT_BUS.post(postEvent);
 				}
 			}
 		}
 	}
-	
+
 //	@SubscribeEvent
 //	public static void onDevilFruitDropped(ItemTossEvent event)
 //	{
```

`src/main/java/xyz/pixelatedw/mineminenomi/items/AkumaNoMiBoxItem.java`

```diff
 import net.minecraft.util.Util;
 import net.minecraft.util.text.TranslationTextComponent;
 import net.minecraft.world.World;
+import net.minecraftforge.common.MinecraftForge;
 import xyz.pixelatedw.mineminenomi.api.OneFruitEntry;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.InventoryDevilFruitEvent;
 import xyz.pixelatedw.mineminenomi.api.helpers.DevilFruitHelper;
 import xyz.pixelatedw.mineminenomi.data.world.ExtendedWorldData;
 import xyz.pixelatedw.mineminenomi.init.ModCreativeTabs;
@@ -86,7 +88,11 @@ public class AkumaNoMiBoxItem extends Item
 				{
 					player.inventory.add(new ItemStack(randomFruit));
 					ExtendedWorldData worldProps = ExtendedWorldData.get(player.level);
+
 					worldProps.updateOneFruit(((AkumaNoMiItem) randomFruit).getFruitKey(), player.getUUID(), OneFruitEntry.Status.INVENTORY, "Obtained from " + itemStack.getDisplayName().getString());
+					InventoryDevilFruitEvent event = new InventoryDevilFruitEvent(player, randomFruit, "Obtained from " + itemStack.getDisplayName().getString());
+					MinecraftForge.EVENT_BUS.post(event);
+
 					return new ActionResult<>(ActionResultType.SUCCESS, player.getItemInHand(hand));
 				}
 			}
```

`src/main/java/xyz/pixelatedw/mineminenomi/items/AkumaNoMiItem.java`

```diff
 import xyz.pixelatedw.mineminenomi.api.abilities.AbilityCore;
 import xyz.pixelatedw.mineminenomi.api.enums.AbilityCommandGroup;
 import xyz.pixelatedw.mineminenomi.api.enums.FruitType;
-import xyz.pixelatedw.mineminenomi.api.events.EatDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.EatDevilFruitEvent;
 import xyz.pixelatedw.mineminenomi.api.helpers.AbilityHelper;
 import xyz.pixelatedw.mineminenomi.api.helpers.ItemsHelper;
 import xyz.pixelatedw.mineminenomi.config.CommonConfig;
@@ -145,13 +145,13 @@ public class AkumaNoMiItem extends Item implements IFruitColor
 	
 			if(CommonConfig.INSTANCE.isYamiPowerEnabled())
 			{
-				// If the player eats any fruit besides yami and it currently doesn't have Yami ate: death
+				// If the player eats any fruit besides yami and they currently don't have Yami eaten: death
 				// ex: mera + pika = death
 				if(hasFruit && eatenItem != ModAbilities.YAMI_YAMI_NO_MI && !hasYami)
 				{
 					this.applyCurseDeath(player);
 					itemStack.shrink(1);
-					worldData.lostOneFruit(eatenFruit, player.getUUID(), "Devil Fruit's Curse");
+					worldData.lostOneFruit(eatenFruit, player, "Devil Fruits Curse");
 					return itemStack;
 				}
 				// If the player eats Yami while already having Yami: death
@@ -160,16 +160,16 @@ public class AkumaNoMiItem extends Item implements IFruitColor
 				{
 					this.applyCurseDeath(player);
 					itemStack.shrink(1);
-					worldData.lostOneFruit(eatenFruit, player.getUUID(), "Devil Fruit's Curse");
+					worldData.lostOneFruit(eatenFruit, player, "Devil Fruits Curse");
 					return itemStack;
 				}
-				// If any fruit is ate while the user has a fruit (different than yami) and yami power: death
+				// If any fruit is eaten while the user has a fruit (different than yami) and yami power: death
 				// ex: pika + yami + anything = death
 				else if((hasFruit && !devilFruitProps.getDevilFruit().equalsIgnoreCase("yami_yami")) && devilFruitProps.hasYamiPower())
 				{
 					this.applyCurseDeath(player);
 					itemStack.shrink(1);
-					worldData.lostOneFruit(eatenFruit, player.getUUID(), "Devil Fruit's Curse");
+					worldData.lostOneFruit(eatenFruit, player, "Devil Fruits Curse");
 					return itemStack;
 				}
 			}
@@ -320,7 +320,8 @@ public class AkumaNoMiItem extends Item implements IFruitColor
 //		}
 
 		boolean shouldRemove = false;
-		
+		String removeReason = null;
+
 		List<BlockPos> blockPosList = WyHelper.getNearbyTileEntities(entity.blockPosition(), entity.level, 2);
 
 		for (BlockPos pos : blockPosList)
@@ -336,14 +337,18 @@ public class AkumaNoMiItem extends Item implements IFruitColor
 
 		List<Entity> hopperMinecarts = WyHelper.getNearbyEntities(entity.blockPosition(), entity.level, 0.5, null, HopperMinecartEntity.class);
 
-		if(hopperMinecarts.size() > 0)
+		if(hopperMinecarts.size() > 0) {
 			shouldRemove = true;
+			removeReason = "Hopper in Minecraft";
+		}
 		
 		List<Entity> foxes = WyHelper.getNearbyEntities(entity.blockPosition(), entity.level, 1.5, null, FoxEntity.class);
 
-		if(foxes.size() > 0)
+		if(foxes.size() > 0) {
 			shouldRemove = true;
-		
+			removeReason = "Fox took fruit";
+		}
+
 		if(shouldRemove) 
 		{
 			entity.remove();
@@ -354,10 +359,10 @@ public class AkumaNoMiItem extends Item implements IFruitColor
 				if(thrower != null)
 					thrower.inventory.add(itemStack);
 				else
-					worldData.updateOneFruit(fruitKey, null, OneFruitEntry.Status.LOST);
+					worldData.lostOneFruit(fruitKey, null, removeReason); 
 			}
 			else
-				worldData.updateOneFruit(fruitKey, null, OneFruitEntry.Status.LOST);
+				worldData.lostOneFruit(fruitKey, null, removeReason);
 		}
 
 		return false;
```

`src/main/java/xyz/pixelatedw/mineminenomi/packets/client/crew/CCreateCrewPacket.java`

```diff
 import net.minecraft.util.text.StringTextComponent;
 import net.minecraft.util.text.TextFormatting;
 import net.minecraft.util.text.TranslationTextComponent;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.fml.network.NetworkDirection;
 import net.minecraftforge.fml.network.NetworkEvent;
 import xyz.pixelatedw.mineminenomi.api.crew.Crew;
+import xyz.pixelatedw.mineminenomi.api.events.CrewEvent;
 import xyz.pixelatedw.mineminenomi.config.CommonConfig;
 import xyz.pixelatedw.mineminenomi.data.entity.entitystats.EntityStatsCapability;
 import xyz.pixelatedw.mineminenomi.data.entity.entitystats.IEntityStats;
@@ -63,21 +65,25 @@ public class CCreateCrewPacket
 				if(!hasSakeBottle || isAlreadyInCrew || !props.isPirate()) {
 					return;					
 				}
-				
 				Crew crew = new Crew(message.name, player);
-				worldProps.addCrew(crew);
-				crew.create(player.level);
-
-				if (CommonConfig.INSTANCE.isCrewWorldMessageEnabled())
+				
+				CrewEvent.Create event = new CrewEvent.Create(player, crew);
+				if(!MinecraftForge.EVENT_BUS.post(event))
 				{
-					TranslationTextComponent newCrewMsg = new TranslationTextComponent(ModI18n.CREW_MESSAGE_NEW_CREW, message.name);
-					for (PlayerEntity target : player.level.players())
+					worldProps.addCrew(crew);
+					crew.create(player.level);
+
+					if (CommonConfig.INSTANCE.isCrewWorldMessageEnabled())
 					{
-						target.sendMessage(new StringTextComponent(TextFormatting.GOLD + newCrewMsg.getString()), Util.NIL_UUID);
+						TranslationTextComponent newCrewMsg = new TranslationTextComponent(ModI18n.CREW_MESSAGE_NEW_CREW, message.name);
+						for (PlayerEntity target : player.level.players())
+						{
+							target.sendMessage(new StringTextComponent(TextFormatting.GOLD + newCrewMsg.getString()), Util.NIL_UUID);
+						}
 					}
+
+					WyNetwork.sendToAll(new SSyncWorldDataPacket(worldProps));
 				}
-				
-				WyNetwork.sendToAll(new SSyncWorldDataPacket(worldProps));
 			});
 		}
 		ctx.get().setPacketHandled(true);
```

`src/main/java/xyz/pixelatedw/mineminenomi/packets/client/crew/CKickFromCrewPacket.java`

```diff
 import net.minecraft.entity.player.PlayerEntity;
 import net.minecraft.network.PacketBuffer;
 import net.minecraft.util.text.TranslationTextComponent;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.fml.network.NetworkDirection;
 import net.minecraftforge.fml.network.NetworkEvent;
+
 import xyz.pixelatedw.mineminenomi.api.crew.Crew;
+import xyz.pixelatedw.mineminenomi.api.events.CrewEvent;
 import xyz.pixelatedw.mineminenomi.api.helpers.FactionHelper;
 import xyz.pixelatedw.mineminenomi.data.world.ExtendedWorldData;
 import xyz.pixelatedw.mineminenomi.init.ModI18n;
@@ -52,11 +55,15 @@ public class CKickFromCrewPacket
 				
 				if(crew != null && crew.hasMember(uuid))
 				{
-					FactionHelper.sendMessageToCrew(sender.level, crew, new TranslationTextComponent(ModI18n.CREW_MESSAGE_KICKED, crew.getMember(uuid).getUsername()));
-					worldData.removeCrewMember(crew, uuid);
-					if(memberPlayer != null)
-						WyNetwork.sendTo(new SSyncWorldDataPacket(worldData), memberPlayer);
-					FactionHelper.sendUpdateMessageToCrew(sender.level, crew);
+					CrewEvent.Kick event = new CrewEvent.Kick(memberPlayer, crew);
+					if(!MinecraftForge.EVENT_BUS.post(event))
+					{
+						FactionHelper.sendMessageToCrew(sender.level, crew, new TranslationTextComponent(ModI18n.CREW_MESSAGE_KICKED, crew.getMember(uuid).getUsername()));
+						worldData.removeCrewMember(crew, uuid);
+						if(memberPlayer != null)
+							WyNetwork.sendTo(new SSyncWorldDataPacket(worldData), memberPlayer);
+						FactionHelper.sendUpdateMessageToCrew(sender.level, crew);
+					}
 				}
 			});	
 		}
```

`src/main/java/xyz/pixelatedw/mineminenomi/packets/client/crew/CUpdateJollyRogerPacket.java`

```diff
 import net.minecraft.entity.player.PlayerEntity;
 import net.minecraft.network.PacketBuffer;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.fml.network.NetworkDirection;
 import net.minecraftforge.fml.network.NetworkEvent;
 import xyz.pixelatedw.mineminenomi.api.crew.Crew;
 import xyz.pixelatedw.mineminenomi.api.crew.JollyRoger;
+import xyz.pixelatedw.mineminenomi.api.events.JollyRogerEvent;
 import xyz.pixelatedw.mineminenomi.data.world.ExtendedWorldData;
 import xyz.pixelatedw.mineminenomi.packets.server.SSyncWorldDataPacket;
 import xyz.pixelatedw.mineminenomi.wypi.WyNetwork;
@@ -49,6 +51,10 @@ public class CUpdateJollyRogerPacket
 				ExtendedWorldData worldData = ExtendedWorldData.get(player.level);
 
 				Crew crew = worldData.getCrewWithCaptain(uuid);
+
+				JollyRogerEvent.Update event = new JollyRogerEvent.Update(message.jollyRoger, crew);
+				MinecraftForge.EVENT_BUS.post(event);
+
 				if (crew != null)
 					worldData.updateCrewJollyRoger(crew, message.jollyRoger);
 
```

`src/main/java/xyz/pixelatedw/mineminenomi/packets/client/ofpw/CRequestCreativeSpawnOFPWPacket.java`

```diff
 				/*
 				 * This packet is send when the item is deleted using the creative-only
 				 * "delete item" slot, this assumed the player is in creative and has creative
-				 * permissions in order to achievee Obviously also assumes the fruit is already
+				 * permissions in order to achieve. Obviously also assumes the fruit is already
 				 * in their inventory otherwise there's no point in deleting it
 				 */
 
@@ -62,8 +62,7 @@ public class CRequestCreativeDeleteOFPWPacket {
 						return;
 					}
 				}
-
-				worldProps.updateOneFruit(message.fruitId, player.getUUID(), OneFruitEntry.Status.LOST, "Deleted using creative inventory");
+				worldProps.lostOneFruit(message.fruitId, player, "Deleted using creative inventory");
 			});
 		}
 	}
```

`src/main/java/xyz/pixelatedw/mineminenomi/packets/client/ofpw/CRequestCreativeSpawnOFPWPacket.java`

```java
 import net.minecraft.entity.player.ServerPlayerEntity;
 import net.minecraft.network.PacketBuffer;
+import net.minecraftforge.common.MinecraftForge;
 import net.minecraftforge.fml.network.NetworkDirection;
 import net.minecraftforge.fml.network.NetworkEvent;
 import xyz.pixelatedw.mineminenomi.api.OneFruitEntry;
+import xyz.pixelatedw.mineminenomi.api.events.onefruit.InventoryDevilFruitEvent;
+import xyz.pixelatedw.mineminenomi.api.helpers.DevilFruitHelper;
 import xyz.pixelatedw.mineminenomi.data.world.ExtendedWorldData;
 
 public class CRequestCreativeSpawnOFPWPacket {
@@ -55,6 +58,8 @@ public class CRequestCreativeSpawnOFPWPacket {
 				}
 
 				worldProps.updateOneFruit(message.fruitId, player.getUUID(), OneFruitEntry.Status.INVENTORY, "Spawned using creative inventory");
+				InventoryDevilFruitEvent event = new InventoryDevilFruitEvent(player, DevilFruitHelper.getDevilFruitItem(message.fruitId), "Spawned using creative inventory");
+				MinecraftForge.EVENT_BUS.post(event);
 			});
 		}
 	}
```

`src/main/java/xyz/pixelatedw/mineminenomi/setup/ForgeSetup.java`

```diff
 package xyz.pixelatedw.mineminenomi.setup;
 
+import java.time.Instant;
 import java.util.HashMap;
 import java.util.Map;
-
 import com.mojang.brigadier.CommandDispatcher;
 import com.mojang.brigadier.builder.LiteralArgumentBuilder;
-
 import net.minecraft.command.CommandSource;
 import net.minecraft.command.Commands;
 import net.minecraft.world.biome.Biome;
@@ -23,6 +22,7 @@ import net.minecraftforge.eventbus.api.SubscribeEvent;
 import net.minecraftforge.fml.common.Mod;
 import net.minecraftforge.fml.event.server.FMLServerStartingEvent;
 import xyz.pixelatedw.mineminenomi.ModMain;
+import xyz.pixelatedw.mineminenomi.api.crew.Crew;
 import xyz.pixelatedw.mineminenomi.commands.AbilityCommand;
 import xyz.pixelatedw.mineminenomi.commands.AbilityProtectionCommand;
 import xyz.pixelatedw.mineminenomi.commands.BellyCommand;
@@ -43,6 +43,7 @@ import xyz.pixelatedw.mineminenomi.commands.QuestCommand;
 import xyz.pixelatedw.mineminenomi.commands.RemoveDFCommand;
 import xyz.pixelatedw.mineminenomi.config.SystemConfig;
 import xyz.pixelatedw.mineminenomi.data.IronValuesManager;
+import xyz.pixelatedw.mineminenomi.data.world.ExtendedWorldData;
 import xyz.pixelatedw.mineminenomi.init.ModEntities;
 import xyz.pixelatedw.mineminenomi.init.ModFeatures;
 import xyz.pixelatedw.mineminenomi.init.ModStructures;
@@ -51,23 +52,21 @@ import xyz.pixelatedw.mineminenomi.wypi.WyPatreon;
 import xyz.pixelatedw.mineminenomi.wypi.WyPatreon.BuildMode;
 
 @Mod.EventBusSubscriber(modid = ModMain.PROJECT_ID, bus = Mod.EventBusSubscriber.Bus.FORGE)
-public class ForgeSetup
-{
+public class ForgeSetup {
 	@SubscribeEvent
 	public static void addReloadListeners(AddReloadListenerEvent event) {
 		event.addListener(IronValuesManager.INSTANCE);
 	}
-	
+
 	@SubscribeEvent
-	public static void serverStarting(FMLServerStartingEvent event)
-	{
+	public static void serverStarting(FMLServerStartingEvent event) {
 		CommandDispatcher dispatcher = event.getServer().getCommands().getDispatcher();
 
 		LiteralArgumentBuilder<CommandSource> masterBuilder = null;
 		boolean masterCommandFlag = SystemConfig.MASTER_COMMAND.get();
-		if(masterCommandFlag)
+		if (masterCommandFlag)
 			masterBuilder = Commands.literal("mmnm");
-		
+
 		AbilityProtectionCommand.register(dispatcher, masterBuilder);
 		DorikiCommand.register(dispatcher, masterBuilder);
 		BountyCommand.register(dispatcher, masterBuilder);
@@ -85,38 +84,46 @@ public class ForgeSetup
 		DamageMultiplierCommand.register(dispatcher, masterBuilder);
 		LoyaltyCommand.register(dispatcher, masterBuilder);
 		GoRogueCommand.register(dispatcher, masterBuilder);
-		
-		if(WyPatreon.BUILD_MODE != BuildMode.RELEASE)
+
+		if (WyPatreon.BUILD_MODE != BuildMode.RELEASE)
 			FGCommand.register(dispatcher, masterBuilder);
-		
-		if(masterCommandFlag)
-			dispatcher.register(masterBuilder);	
+
+		if (masterCommandFlag)
+			dispatcher.register(masterBuilder);
+
+		// Migrate existing Crews
+		for (Crew crew : ExtendedWorldData.get().getCrews()) {
+			if (crew.getCreationDate() == 0L) {
+				crew.setCreationDate(Instant.now().getEpochSecond());
+				ExtendedWorldData.get().setDirty();
+			}
+		}
 	}
 
 	@SubscribeEvent(priority = EventPriority.HIGH)
-	public static void biomeModification(BiomeLoadingEvent event)
-	{
-		if(event.getCategory() == Biome.Category.NETHER || event.getCategory() == Biome.Category.THEEND)
+	public static void biomeModification(BiomeLoadingEvent event) {
+		if (event.getCategory() == Biome.Category.NETHER
+				|| event.getCategory() == Biome.Category.THEEND)
 			return;
-		
-		// Add our structure to all biomes including other modded biomes, we individually check for the biomes and their specs for each structure if they're OPStructure
-		for(Map.Entry<Structure<?>, StructureFeature<?, ?>> entry : ModStructures.REGISTERED_STRUCTURES.entrySet())
-		{
-			if(entry.getKey() instanceof OPStructure && !((OPStructure)entry.getKey()).biomeCheck(event))
+
+		// Add our structure to all biomes including other modded biomes, we individually check for
+		// the biomes and their specs for each structure if they're OPStructure
+		for (Map.Entry<Structure<?>, StructureFeature<?, ?>> entry : ModStructures.REGISTERED_STRUCTURES
+				.entrySet()) {
+			if (entry.getKey() instanceof OPStructure
+					&& !((OPStructure) entry.getKey()).biomeCheck(event))
 				continue;
 			event.getGeneration().getStructures().add(() -> entry.getValue());
 		}
-		
+
 		ModFeatures.setupFeatures(event);
-		
+
 		ModEntities.setupCategorySpawns(event);
 	}
-	
+
 	@SubscribeEvent
-	public static void addDimensionalSpacing(WorldEvent.Load event)
-	{
-		if (event.getWorld() instanceof ServerWorld)
-		{
+	public static void addDimensionalSpacing(WorldEvent.Load event) {
+		if (event.getWorld() instanceof ServerWorld) {
 			ServerWorld serverWorld = (ServerWorld) event.getWorld();
 
 			// Prevent spawning our structure in Vanilla's superflat world as
@@ -125,19 +132,20 @@ public class ForgeSetup
 			if (serverWorld.getChunkSource().getGenerator() instanceof FlatChunkGenerator)
 				return;
 
-			Map<Structure<?>, StructureSeparationSettings> tempMap = new HashMap<>(serverWorld.getChunkSource().getGenerator().getSettings().structureConfig);
-			for(Map.Entry<Structure<?>, StructureFeature<?, ?>> entry : ModStructures.REGISTERED_STRUCTURES.entrySet())
-				tempMap.put(entry.getKey(), DimensionStructuresSettings.DEFAULTS.get(entry.getKey()));
+			Map<Structure<?>, StructureSeparationSettings> tempMap = new HashMap<>(
+					serverWorld.getChunkSource().getGenerator().getSettings().structureConfig);
+			for (Map.Entry<Structure<?>, StructureFeature<?, ?>> entry : ModStructures.REGISTERED_STRUCTURES
+					.entrySet())
+				tempMap.put(entry.getKey(),
+						DimensionStructuresSettings.DEFAULTS.get(entry.getKey()));
 			serverWorld.getChunkSource().getGenerator().getSettings().structureConfig = tempMap;
 		}
 	}
-	
+
 	/*
-	@SubscribeEvent
-	public static void onRegisterDimensionsEvent(RegisterDimensionsEvent event)
-	{
-		if (DimensionType.byName(ModResources.DIMENSION_TYPE_CHALLENGES) == null)
-			DimensionManager.registerDimension(ModResources.DIMENSION_TYPE_CHALLENGES, ModDimensions.CHALLENGES, null, true);
-	}
-	*/
-}
\ No newline at end of file
+	 * @SubscribeEvent public static void onRegisterDimensionsEvent(RegisterDimensionsEvent event) {
+	 * if (DimensionType.byName(ModResources.DIMENSION_TYPE_CHALLENGES) == null)
+	 * DimensionManager.registerDimension(ModResources.DIMENSION_TYPE_CHALLENGES,
+	 * ModDimensions.CHALLENGES, null, true); }
+	 */
+}

```